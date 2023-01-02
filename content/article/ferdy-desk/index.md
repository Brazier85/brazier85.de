---
title: "Ferdy Deck"
date: 2023-01-01T15:00:00+01:00
draft: false

categories: ["code"]
tags: ["Home Assistant", "SmartHome", "FerdyDesk", "Streamdeck"]
toc: false
author: "Ferdinand Berger"
---

Recently I stumbled over a article on hackaday which was related to a project from David Zhang on [github](https://github.com/davidz-yt/desk-controller). He made a desk-controller - a small budged - stream desk.

<!--more-->

## Required components

- [LogiLink ID0120](https://amzn.to/3VyIjYc)
- [Raspberry Pi Pico](https://amzn.to/3VyIi6A)
- [USB extension cable](https://amzn.to/3vA2TNo)

**Note:** The links above are amazon affiliate links

The whole setup makes use of the HID-remapper firmware for the Raspberry Pi Pico found [here](https://github.com/jfedor2/hid-remapper) and [AutoHotkey](https://www.autohotkey.com/).

Basically you remap the keys from your external Keypad to something like `shift + F15` which then will be captured by a AutoHotkey-script that runs several actions.

## HID-remapper

For the baisc setup on HID-remapper setup please visit this page here on [github](https://github.com/jfedor2/hid-remapper/blob/master/HARDWARE.md). Basicaly you have to start the Raspberry Pi in boot mode, copy the firmware file over and then use this [website](https://www.jfedor.org/hid-remapper-config/) to configure your hotkeys.

Here is my configuration:
{{< code type="json" title="hid-remapper-config.json" >}}
{
    "version": 3,
    "unmapped_passthrough": false,
    "partial_scroll_timeout": 1000000,
    "interval_override": 0,
    "mappings": [
        {
            "target_usage": "0x000700e1",
            "source_usage": "0x00070062",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x00070068",
            "source_usage": "0x00070062",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e1",
            "source_usage": "0x00070063",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x00070069",
            "source_usage": "0x00070063",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e1",
            "source_usage": "0x00070058",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x0007006a",
            "source_usage": "0x00070058",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e1",
            "source_usage": "0x00070059",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x0007006b",
            "source_usage": "0x00070059",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e1",
            "source_usage": "0x0007005a",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x0007006c",
            "source_usage": "0x0007005a",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e1",
            "source_usage": "0x0007005b",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x0007006d",
            "source_usage": "0x0007005b",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e1",
            "source_usage": "0x0007005c",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x0007006e",
            "source_usage": "0x0007005c",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e1",
            "source_usage": "0x0007005d",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x0007006f",
            "source_usage": "0x0007005d",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e1",
            "source_usage": "0x0007005e",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x00070070",
            "source_usage": "0x0007005e",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e1",
            "source_usage": "0x00070057",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x00070071",
            "source_usage": "0x00070057",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e1",
            "source_usage": "0x0007005f",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x00070072",
            "source_usage": "0x0007005f",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e1",
            "source_usage": "0x00070060",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x00070073",
            "source_usage": "0x00070060",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e2",
            "source_usage": "0x00070061",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x00070068",
            "source_usage": "0x00070061",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e2",
            "source_usage": "0x00070056",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x00070069",
            "source_usage": "0x00070056",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e2",
            "source_usage": "0x00070053",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x0007006a",
            "source_usage": "0x00070053",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e2",
            "source_usage": "0x00070054",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x0007006b",
            "source_usage": "0x00070054",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e2",
            "source_usage": "0x00070055",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x0007006c",
            "source_usage": "0x00070055",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x000700e2",
            "source_usage": "0x0007002a",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        },
        {
            "target_usage": "0x0007006d",
            "source_usage": "0x0007002a",
            "scaling": 1000,
            "layer": 0,
            "sticky": false
        }
    ]
}
{{< /code >}}

## AutoHotkey

I use version 1.1 of AutoHotkey for this script since I was not able to port everything over to version 2. You can get [AutoHotkey](https://www.autohotkey.com/) on there website. In addition to AHK I use the following software:

- [FancyZones](https://learn.microsoft.com/de-de/windows/powertoys/fancyzones)
- [HomeAssistant](/article/basic-smarthome-setup)

{{< code type="ahk" title="FerdyDesk.ahk" >}}
#NoEnv  ; Recommended for performance and compatibility with future AutoHotkey releases.
; #Warn  ; Enable warnings to assist with detecting common errors.
SendMode Input  ; Recommended for new scripts due to its superior speed and reliability.
SetWorkingDir %A_ScriptDir%  ; Ensures a consistent starting directory.

; Note the following hotkey prefixes (^ Ctrl) (! Alt) (+ Shift)
; HID Remapper from https://www.jfedor.org/hid-remapper-config/

global hass_url := "https://zxxxxxxxxx.ui.nabu.casa/api/"
global hass_token := "Bearer eyxxxxxxxxxxxeyJpc3xxxxxxxoxOTg3NzA1MDxxxxxxxxxhE0S2L0xxx"

Sysget, totalWidth, 78
Sysget, totalHeight, 79
global m_width := totalWidth
global m_height := totalHeight

Return

; ############# Quick Launchers #############

; Keypad 0
Shift & F13::
    ; Window left
    WinGetPos, X, Y, W, H, A
    WinMove, A,,0,0,W,H
    SendInput, #{UP}
return

; Keypad .
Shift & F14::
    ; code here
return

; Keypad Enter
Shift & F15::
    ; Mute
    ; Make use of PowerToys VCM
    SendInput, #+a
return

; Keypad 1
Shift & F16::
    ; Meeting
    HASS_Request("POST", "events/streamdeck", "{""event"":""meeting""}")
return

; Keypad 2
Shift & F17::
    ; Zocken
    HASS_Request("POST", "events/streamdeck", "{""event"":""zocken""}")
return

; Keypad 3
Shift & F18::
    ; DELL KVM switch
    SendInput, ^!p
return

; Keypad 4
Shift & F19::
    ; code here
return

; Keypad 5
Shift & F20::
    ; code here
return

; Keypad 6
Shift & F21::
    ; code here
return

; Keypad +
Shift & F22::
    ; Window to center
    WinGetPos, X, Y, W, H, A
    WinMove, A,,(m_width/2)-(W/2),(m_height/2)-(H/2),W,H
return

; Keypad 7
Shift & F23::
    ; Window left
    WinGetPos, X, Y, W, H, A
    WinMove, A,,0,0,W,H
    SendInput, #{UP}
return

; Keypad 8
Shift & F24::
    ; Window center
    WinGetPos, X, Y, W, H, A
    WinMove, A,,(m_width/2)-(W/2),0,W,H
    SendInput, #{UP}
return

; Keypad 9
Alt & F13::
    ; Window right
    WinGetPos, X, Y, W, H, A
    WinMove, A,,(m_width-W),0,W,H
    SendInput, #{UP}
return

; Keypad -
Alt & F14::
    ; Window small
    WinGetPos, X, Y, W, H, A
    WinMove, A,,X+(W/2)-600,Y+(H/2)-500,1200,1000
return

; Keypad Num Lock
Alt & F15::
    ; Fancy Zones Layout 0
    SendInput, ^#!0
    SendInput, #{UP}
return

; Keypad /
Alt & F16::
    ; Fancy Zones Layout 1
    SendInput, ^#!1
    SendInput, #{UP}
return

; Keypad *
Alt & F17::
    ; Open Explorer
    Run, Explorer.exe
return

; Keypad Backspace
Alt & F18::
    Process, Exist, chrome.exe 
    If Not ErrorLevel {
        Run, C:\Program Files\Google\Chrome\Application\chrome.exe 
    } Else {
        WinActivate, ahk_exe chrome.exe
    }
return

; 2023
:b0?*:2022::
    TrayTip Remember`,, it's 2023 now!
return

; Functions
HASS_Request(http_method, target, data) {

    oHTTP := ComObjCreate("WinHttp.WinHttpRequest.5.1")
    if !IsObject(oHTTP) {
        MsgBox,4112,Fatal Error,Unable to create HTTP object
        return
    }

    oHTTP.open(http_method, hass_url . target)
    oHTTP.SetRequestHeader("Authorization", hass_token)
    oHTTP.setRequestHeader("Content-Type", "application/json")
    oHTTP.Send(data)
    return oHTTP.ResponseText
}

; Script end
{{< /code >}}

## Known issues

At the moment I have the following problems detected with this setup:

- If you press the buttons to fast it does not work properly
- Sometimes the `Ctrl` or `Alt`key seem to be stuck...

If you have any ideas how to fix this issues please let me know :)