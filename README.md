# Intune Guide: Securing Wi-Fi Passwords on Windows Devices

## Overview
This document outlines the configuration strategies available in Microsoft Intune to prevent users from viewing or retrieving Wi-Fi passwords (Pre-Shared Keys) on managed Windows 10/11 devices.

**Target Scenario:**
*   **Network Type:** WPA/WPA2/WPA3-Personal (PSK).
*   **Challenge:** Personal networks require the password to be cached on the device to function. Therefore, "Remember Credentials" cannot be disabled.
*   **Goal:** Prevent the user from viewing the cached password via the Windows UI or Command Line.

---

## Strategy 1: Deploy Wi-Fi Profile via Intune (Foundation)
**Why:** By pushing the profile via Intune, the user never needs to type the password manually. If they don't type it, they don't know it. Additionally, Intune-managed profiles often lock the "Properties" UI by default, preventing the password from being viewed.

**Configuration Steps:**
1.  Navigate to **Devices** > **Windows** > **Configuration profiles**.
2.  Click **Create** > **New Policy**.
3.  Platform: **Windows 10 and later**.
4.  Profile type: **Templates** > **Wi-Fi**.
5.  Policy Name: `WIN - DC - Wi-Fi Profile - Office Wi-Fi Network`
6.  Wi-Fi Type: **Basic**.
7.  Wi-Fi Name (SSID): **Enter the exact SSID of your network**.
8.  Connection Name: **Enter a user-friendly name (e.g., "Corporate Wi-Fi")**.
9.  Connect automatically: **Yes**.
10. Wireless Security Type: **WPA/WPA2-Personal**.
11. Pre-shared key (PSK): **Enter the Wi-Fi password here**.
12. **Assignment:** Assign to a **Device Group**.
*Note: Assigning to a Device Group ensures the device connects to the Wi-Fi network at the login screen, before a user logs in.*

---

## Strategy 2: Hide the Wi-Fi Settings Page (Recommended)
**Why:** This removes the "Wi-Fi" and "Network Status" pages from the Windows Settings app. If the user cannot access the page, they cannot click the "View Wi-Fi Security Key" or "Show" button. This is the most reliable method for Windows 11 24H2+.

**Configuration Steps (Settings Catalog):**
1.  Navigate to **Devices** > **Windows** > **Configuration profiles**.
2.  Click **Create** > **New Policy**.
3.  Platform: **Windows 10 and later**.
4.  Profile type: **Settings catalog**.
5.  Name the profile: `WIN - SC - Device Security - Hide Wi-Fi Network Settings`
6.  Click **Add settings** and search for **"Page Visibility"**.
7.  Select: **Settings** > **Page Visibility List** (or *Page Visibility List (User)*).
8.  Enable the setting. In the **Settings Page Visibility** text box, enter the following string exactly:
    ```text
    hide:network-wifi;network-status
    ```
9.  **Assignment:** Assign to a **User Group**.
    *Note: Assigning to a User Group blocks regular users from seeing the Network Settings, but allows you (the IT Admin) to log in with your admin account and still see the settings. If assigned to a Device Group, the settings page will be hidden for everyone, including IT Admins.*

---

## Strategy 3: OMA-URI Policy (Deprecated / Not Recommended)
> **Warning:** This method is currently **deprecated** and **not recommended**.
> *   It is known to fail with **Error Code -2016281112** (Remediation Failed) on many Windows Editions (specifically Windows Pro).
> *   It is **not compatible** with the new Windows 11 24H2 network UI, which includes a QR code feature that this policy does not hide.

**Why:** This is the specific Windows Configuration Service Provider (CSP) setting designed to disable the password view button. It is included here only for documentation purposes.

**Configuration Steps:**
1.  Navigate to **Devices** > **Windows** > **Configuration profiles**.
2.  Click **Create** > **New Policy**.
3.  **Platform:** Windows 10 and later.
4.  **Profile type:** Templates > **Custom**.
5.  Click **Add** to create a row:
    *   **Name:** `WIN - DC - (OMA-URI) - DEV - Disable Show Wi-Fi Password`
    *   **OMA-URI:** `./Device/Vendor/MSFT/Policy/Config/Connectivity/DoNotViewWiFiPassword`
    *   **Data type:** Integer
    *   **Value:** `1`

---

## Summary of Options

| Option | Method | Description | Pros | Cons |
| :--- | :--- | :--- | :--- | :--- |
| **1** | **Deploy Wi-Fi Profile** | Push the SSID and Password via Intune. | User never types the password; seamless connection. | Does not strictly block "viewing" if the user is savvy and has Admin rights. |
| **2** | **Hide Wi-Fi Network Settings Page** | Use `hide:network-wifi` in Settings Catalog. | Native, reliable, blocks the UI button effectively on all Windows versions. | Hides other Wi-Fi settings (like MAC randomization). |
| **3** | **OMA-URI CSP** | Use `DoNotViewWiFiPassword` CSP. | The "official" policy for this specific button. | **Unreliable.** Frequently errors with `-2016281112`. Does not work on Windows 11 24H2+. |
