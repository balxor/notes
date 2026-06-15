# RE ByteDance GP Anti-Cheat — Ragnarok Online SEA

## ✅ Completed (Analysis Phase)

### Arsitektur Anti-Cheat
- [x] Identifikasi: anti-cheat adalah **ByteDance GP** (Game Protection), **bukan** nProtect GameGuard
- [x] Komponen ditemukan di:
  - `C:\RO_SEA\RO_SEAGame\gpHackerProc.dll` (3.5MB, v2.3.1) — main anti-cheat, **OpenSSL 3.1.2 statis**, **di-pack/obfuscated**
  - `C:\RO_SEA\RO_SEAGame\gpShell.dll` (384KB, v2.3.1) — GP shell loader
  - `C:\RO_SEA\RO_SEAGame\Plugins\x86_64\gp.dll` (6.7MB, v2.5.1) — GP native Unity plugin
  - `C:\RO_SEA\RO_SEAGame\Plugins\x86_64\gpm.dll` (287KB) — Game Performance Monitoring
  - `C:\RO_SEA\RO_SEAGame\Plugins\x86_64\gpmperf.dll` (240KB) — Performance profiler
  - `C:\RO_SEA\RO_SEAGame\Plugins\x86_64\gsdk.dll` (2.4MB, v3.23.0) — ByteDance Game SDK inti
  - `C:\RO_SEA\RO_SEAGame\Plugins\x86_64\parfait.dll` (1MB) + `parfait_crash_handler.exe` — Crash reporter

### Config & Konfigurasi
- [x] `C:\RO_SEA\RO_SEAGame\config.xml` — fitur confirm:
  - `<FileVerification forceVerify="0">` — integrity check file (saat ini nonaktif)
  - `<inject>` — GP bisa inject DLL sendiri ke game process
  - `<SignatureBanList>` — daftar signature cheat tool yang di-ban
- [x] `C:\RO_SEA\RO_SEAGame\Plugins\x86_64\config.json` — game_id: `3402`, app_name: `rosea`, server_region: 10
- [x] `C:\RO_SEA\RO_SEAGame\StreamingAssets\Assembly-CSharp.dhao.bytes` — MD5: `81431E5EB49FCE911BAEEDA602A0FE47` (file integrity hash)
- [x] `C:\RO_SEA\RO_SEAGame\StreamingAssets\manifest.txt` — konfirmasi Assembly-CSharp.dll + hash

### C# Bridge (dump.cs)
- [x] `C:\RO_SEA\il2cpp_output\dump.cs` — semua P/Invoke ke native anti-cheat ditemukan:

| Fungsi | Line dump.cs | Native Native Address | Fungsi |
|---|---|---|---|
| `SecurityService.IsGPDllExist()` | 482929 | `GameAssembly.dll` 0x160A710 | Cek apakah GP DLL ter-load |
| `SecurityService.DisableGP()` | 482932 | `GameAssembly.dll` 0x160A580 | **Nonaktifkan GP** |
| `GNAMonitorBegin(target, extraInfo)` | 482127 | `GameAssembly.dll` 0x1F1A790 | Mulai monitor target |
| `GNAMonitorEnd(extraInfo)` | 482130 | `GameAssembly.dll` 0x1F1A840 | Stop monitor |
| `GNADoDiagnosisDuringGaming(msg)` | 482133 | `GameAssembly.dll` 0x1F1A700 | Diagnosis saat gameplay |
| `GPSetUserInfo(roleId, serverId)` | 482938 | `GameAssembly.dll` 0x160A5F0 | Set user info ke GP |
| `GSDKIsGPDllExist()` | 482941 | `GameAssembly.dll` 0x160A710 | Native check (wrapper) |
| `GSDKDisableGP()` | 482944 | `GameAssembly.dll` 0x160A580 | Native disable (wrapper) |

### Data Tambahan
- [x] `gsdk.dll` mengandung full **OpenSSL 3.1.2** statis (build path: `C:\Users\Admin\Downloads\openssl\openssl-openssl-3.1.2\`)
- [x] `gpHackerProc.dll` di-pack — section names acak/terenkripsi, tidak ada API string dalam plaintext
- [x] Version: GSDK `3.23.0`, GP C++ SDK `2.3.1`

---

## 🔜 Next Actions (Prioritized)

### ⚡ HIGH PRIORITY

#### 1. Static RE: `C:\RO_SEA\RO_SEAGame\gpHackerProc.dll`
- [ ] **Unpack/unobfuscate** `.text` section (currently scrambled)
- [ ] **Identifikasi TLS callbacks** & entry point (`DllMain`)
- [ ] **Temukan fungsi deteksi:**
  - `CreateRemoteThread` / `NtCreateThreadEx` — injection detection
  - `OpenProcess` / `NtOpenProcess` — process enumeration
  - `NtQuerySystemInformation` — process list scanning
  - `IsDebuggerPresent` / `CheckRemoteDebuggerPresent` — debugger detection
  - `NtQueryInformationProcess` — ProcessDebugPort, ProcessDebugFlags
  - `WriteProcessMemory` / `VirtualProtectEx` — memory patch detection
  - `GetForegroundWindow` / `EnumWindows` — window title scanning (Cheat Engine, etc.)
- [ ] **Buat detour/patch** untuk fungsi deteksi utama (ret-nop)

#### 2. Patch Native: `C:\RO_SEA\RO_SEAGame\GameAssembly.dll`
- [ ] Buka di **IDA Pro 8.x** atau **Ghidra**
- [ ] Cari fungsi `IsGPDllExist` (native addr: `0x160A710`) — patch:
  ```asm
  xor al, al   ; return false
  ret
