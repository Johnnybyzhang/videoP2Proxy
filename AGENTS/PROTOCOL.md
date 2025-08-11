# Xiaomi P2P Protocol Notes

These notes summarize how `videop2proxy` talks to Xiaomi cameras. They are meant to aid refactoring the proxy in languages such as Python, Go or Rust.

## Credential Discovery
1. The camera's miIO token and IP address are required.
2. Run the following miIO command to obtain P2P credentials:
   ```sh
   python3 -c "import miio; result = miio.device.Device('<IP>', '<TOKEN>').send('get_ipcprop', ['all']); print(result['p2p_id']); print(result['avID']); print(result['avPass']);"
   ```
   This yields the camera's `p2p_id` (UID), AV username and password.

## Connection Sequence
1. `IOTC_Initialize2(0)` – initialise the IOTC stack.
2. `avInitialize(3)` – initialise AV client support.
3. `SID = IOTC_Get_SessionID()` – reserve a session slot.
4. `IOTC_Connect_ByUID_Parallel(p2p_id, SID)` – open a P2P link to the camera.
5. `avClientStart2(SID, userName, passWord, 5000, &srvType, 0, &bResend)` – authenticate and obtain an `avIndex`.
6. Start streaming by sending two IO control messages using `avSendIOCtrl` with payload `{"channel":0}`:
   - `IOTYPE_USER_IPCAM_START`
   - `IOTYPE_USER_IPCAM_AUDIOSTART`
7. Frames are received with `avRecvFrameData2(avIndex, ...)`.
8. To stop streaming send `IOTYPE_USER_IPCAM_STOP` and `IOTYPE_USER_IPCAM_AUDIOSTOP`.

## Frame Metadata Layout
Each `avRecvFrameData2` call returns raw H.264 data and a 64‑byte little‑endian metadata block. Important fields:

| Offset | Size | Field | Description |
|-------:|-----:|-------|-------------|
| 0      | 2    | `codec_id` | Expected to be H264 (`0x6800`). |
| 2      | 1    | `flags` | `1` indicates an I‑frame. |
| 3      | 1    | `cam_index` | Camera index. |
| 4      | 1    | `onlineNum` | Number of online viewers. |
| 5      | 1    | `isLiveflages` | <150 means live, otherwise playback. |
| 6      | 1    | `backwardIndex` |
| 7      | 1    | `backwardIndex2` |
| 8      | 4    | `millisecond` |
| 12     | 4    | `timestamp` | Seconds since epoch. Combine with `millisecond` for frame time. |
| 16     | 4    | `videoWidth` |
| 20     | 4    | `videoHeight` |

The frame payload immediately follows the metadata and its length is returned separately (`frmSize`).

## Output Modes
* **STDOUT** – raw H.264 frames are written to standard output when `--stdout` is supplied.
* **RTSP** – if compiled with `ENABLE_RTSP`, frames are written to a FIFO consumed by a live555‑based server (`rtsp.cxx`).

## Miscellaneous
* Audio support is stubbed out (`thread_ReceiveAudio`), but IO control messages for audio are issued for completeness.
* The project relies on vendor libraries found in `lib/` and headers under `include/`.

These details mirror the behaviour of the current C implementation and can serve as a reference when porting the proxy to other languages.
