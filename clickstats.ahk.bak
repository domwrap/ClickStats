#NoEnv
#SingleInstance Force

; Maketh the system area more friendly
Menu, Tray, Icon, clickstats.ico
Menu, Tray, Tip, Click Stats

; Math thanks to https://gist.github.com/jNizM/5867684#file-gistfile1-ahk

; var config
intStatClicks := 0
intStatKeys := 0
intTimeClicks := 0
intTimeKeys := 0
intTimeSpaces := 0
; Distance setup
intStatDistance := 0
intTimeDistance := 0
MouseGetPos, xPosOld, yPosOld
intTime := A_TickCount
; Timestamps
FormatTime, strDate,, yyyy-MM-dd
FormatTime, strTime,, HH:mm
; @todo read last line and check timestamp
IniRead, strTimeLastLogged, %strFileStats%, Log, Time, %strTime%
IniRead, strDateLastLogged, %strFileStats%, Log, Date, %strDate%

; Logging setup
;FileCreateDir, %A_AppDataCommon%\ClickStats\
;strFileConf = %A_AppDataCommon%\ClickStats\clickstats.ini
FileCreateDir, %A_AppData%\ClickStats\
strFileStats = %A_AppData%\ClickStats\clickstats.ini
; Make start at boot
FileCreateShortcut, %A_ScriptFullPath%, %A_Startup%\ClickStats.lnk,%A_ScriptDir%,,,,,,7
; Make Start Menu group and shortcut
FileCreateDir, %A_ProgramsCommon%\ClickStats
FileCreateShortcut, %A_ScriptFullPath%, %A_ProgramsCommon%\ClickStats\ClickStats.lnk,%A_ScriptDir%,,,,,,7
; Make stats shortcut
; @todo Create startmenu shortcut to stats log

; Start stats at load?
IniRead, blnStats, %strFileStats%, Config, Autostart, True
if ( !blnStats OR ( blnStats != False AND blnStats != "False" ) ) {
	blnStats := True
}

; screen setup
SysGet, intScreenWidth, 78
SysGet, intScreenHeight, 79
intScreenDiagonal := Round(Sqrt( (intScreenWidth/A_ScreenDPI)**2 + (intScreenHeight/A_ScreenDPI)**2 ),2)
;IniRead, intScreenDiagonal, %strFileStats%, Config, ScreenDiagonal, %intScreenDiagonalCalculated%


;intPPI := Sqrt(intScreenWidth **2 + intScreenHeight **2) / intScreenDiagonal
;intPPCm := Sqrt(intScreenWidth **2 + intScreenHeight **2) / (intScreenDiagonal * 2.54)
; Or much easier
intPPI := A_ScreenDPI
intPPCm := A_ScreenDPI / 2.54


; window position
SysGet, intResX, 16
intGuiW = 133
intGuiX := intResX - intGuiW
intIconX := intGuiW - 26
; build the gui
Gui, Add, Button, vbtnStatStart xm w35 gfnStatStart, Star&t
Gui, Add, Button, vbtnStatStop xp+40 wp gfnStatStop, Sto&p
Gui, Add, Button, vbtnStatReset xp+40 wp gfnStatReset, R&eset
Gui, Add, Text, xm, Clicks
Gui, Add, Text, vlblStatClicks xp+64 w50 Right, 0
Gui, Add, Text, xm, Distance
Gui, Add, Text, vlblStatDist xp+64 w50 Right, 0
Gui, Add, Text, xm, Keys
Gui, Add, Text, vlblStatKeystrokes xp+64 w50 Right, 0
Gui, Add, StatusBar, vlblStatusBar, Screen Size: %intScreenDiagonal%"
; setup
GuiControl, Disable, btnStatStop
GuiControl, Disable, btnStatReset
; Make it be!
Gui, Show, x%intGuiX% y100 w%intGuiW% Minimize, Click Stats
WinSet, AlwaysOnTop, On, A
;WinMinimize, A


; Add timer for updates
SetTimer, statUpdate, 30
setTimer, logUpdate, 60000
return


; LABELS ========================================================
statUpdate:
	; DLL call ensures mouse-distance isn't recorded when Windows "switches desktops" (un)locking the screen
	if ( true = blnStats AND DllCall("User32\OpenInputDesktop","int",0*0,"int",0*0,"int",0x0001L*1) ) {
		GuiControl, Disable, btnStatStart
		GuiControl, Enable, btnStatStop
		GuiControl, Enable, btnStatReset

		MouseGetPos, xPosNew, yPosNew
		Diff := % GetPointsDistance(xPosOld, yPosOld, xPosNew, yPosNew)
		intStatDistance += Diff
		intTimeDistance += Diff
		Time2 := A_TickCount

		xPosOld := xPosNew
		yPosOld := yPosNew
	} else {
		GuiControl, Enable, btnStatStart
		GuiControl, Disable, btnStatStop
		if ( 1 > intStatClicks AND 1 > intStatDistance AND 1 > intStatKeys ) {
			GuiControl, Disable, btnStatReset
		}
	}

	; update click stats
	intStatDistCM = % Round((intStatDistance / intPPCm), 0)
	intStatDistM = % Round((intStatDistance / intPPCm / 100), 2)
	intStatDistKM = % Round((intStatDistance / intPPCm / 100000 ), 2)
	if ( 1 > intStatDistM ) {
		GuiControl,, lblStatDist, % intStatDistCM . "cm"
	} else if ( 1 > intStatDistKM ) {
		GuiControl,, lblStatDist, % intStatDistM . "m"
	} else {
		GuiControl,, lblStatDist, % intStatDistKM . "km"
	}
	; update distance stats
	GuiControl,, lblStatClicks,%intStatClicks%
	GuiControl,, lblStatKeystrokes,%intStatKeys%
return


; label to update log timer (based on minute timer)
logUpdate:
	FormatTime, strDate,, yyyy-MM-dd
	FormatTime, strTime,, HH:mm

	if ( strTime != strTimeLastLogged ) {
		strFileLog = %A_AppData%\ClickStats\%A_YYYY%-%A_MM%.log

		; Create log file (with header row) if not already
		IfNotExist, %strFileLog%
		{
			FileAppend, date`,time`,logging`,clicks`,distance`,keys`,spaces`n, %strFileLog%
		}

		; write to log
		strLogLine = %strDate%,%strTime%,%blnStats%,%intTimeClicks%,%intTimeDistance%,%intTimeKeys%,%intTimeSpaces%`n
		FileAppend, %strLogLine%, %strFileLog%
		If Not ErrorLevel
		{
			; Timestamp the writing
			FormatTime, strDateLastLogged,, yyyy-MM-dd
			FormatTime, strTimeLastLogged,, HH:mm
			; Write last logged to conf
			IniWrite, %strDateLastLogged%, %strFileStats%, Log, Date
			IniWrite, %strTimeLastLogged%, %strFileStats%, Log, Time

			; reset vars
			intTimeClicks = 0
			intTimeKeys = 0
			intTimeSpaces = 0
			intTimeDistance = 0
		}
	}
return


fnStatStart:
	blnStats := true
return


fnStatStop:
	blnStats := false
	if ( 0 < intStatClicks )
		intStatClicks--
return


fnStatReset:
	; Stats
	IniRead, intSavedClicks, %strFileStats%, Stats, Clicks, 0
	IniRead, intSavedDistance, %strFileStats%, Stats, Distance, 0
	IniRead, intSavedKeystrokes, %strFileStats%, Stats, Keystrokes, 0
	IniWrite, % intSavedClicks + intStatClicks, %strFileStats%, Stats, Clicks
	IniWrite, % intSavedDistance + intStatDistance, %strFileStats%, Stats, Distance
	IniWrite, % intSavedKeystrokes + intStatKeys, %strFileStats%, Stats, Keystrokes

	; reset vars
	blnStats := false
	intStatClicks = 0
    intStatDistance := 0
    intStatKeys := 0
    intTime := A_TickCount
return

fnExitSave:
	; Reset last logged to force write (don't lose stats if closing in same minute as last write)
	strTimeLastLogged := 0
	; update logs
	Gosub, logUpdate
	; write total stats
	Gosub, fnStatReset
	; write conf
	IniWrite, %intScreenWidth%, %strFileStats%, Conf, ScreenWidth
	IniWrite, %intScreenHeight%, %strFileStats%, Conf, ScreenHeight
	IniWrite, %A_ScreenDPI%, %strFileStats%, Conf, ScreenDPI
	IniWrite, %intScreenDiagonal%, %strFileStats%, Conf, ScreenDiagonal

	; KILL!
	ExitApp
return

; FUNCTIONS ========================================================
; math to work out distance travelled
GetPointsDistance(x1, y1, x2, y2) {
    return, Sqrt(((x1 - x2) **2)  + ((y1 - y2) **2))
}


; @todo minutely logging of statistics

; @todo count spaces separately (keys/spaces=words per minute)

; @todo add graphing?
; http://www.autohotkey.com/board/topic/64563-basic-ahk-v11-com-tutorial-for-webpages/
;
; http://www.amcharts.com/download/
; http://www.joellipman.com/articles/automation/autohotkey/514-basic-webpage-controls-with-javascript--com.html
; http://msdn.microsoft.com/en-us/library/ie/12k71sw7

; @todo Use database (sqlite?)
; http://www.autohotkey.com/board/topic/71179-ahk-l-dba-16-oop-sql-database-sqlite-mysql-ado/
; http://www.sqlite.org/
; http://www.autohotkey.com/board/topic/67427-class-sqlitedb-now-with-blob-and-u64-support/

; @todo Log per-app statistics?  Lots more work for that
; Could collect a per-process array of clicks/keys/mouse and just remove the timeframe element
; So you can report on clicks over time OR by process
; Not both as that would require recording every single event


; CLOSING ========================================================
OnExit, fnExitSave


GuiClose:
	Gosub, fnExitSave



; HOTKEYS ========================================================

; Capture click event for stat counting
~*LButton::
~*RButton::
~*MButton::
	If ( true = blnStats ) {
		intTimeClicks++
		intStatClicks++
	}
; @todo record individual click with
; - absolute coords
; - process
; - relative coords
; - timestamp? (maybe)
return


; Capture spaces separately
~SC039::
	If ( true = blnStats ) {
		intTimeSpaces++
		intStatKeys++
	}
return

; Capture keys
~SC001::
~SC002::
~SC003::
~SC004::
~SC005::
~SC006::
~SC007::
~SC008::
~SC009::
~SC00a::
~SC00b::
~SC00c::
~SC00d::
~SC00e::
~SC00f::
~SC010::
~SC011::
~SC012::
~SC013::
~SC014::
~SC015::
~SC016::
~SC017::
~SC018::
~SC019::
~SC01a::
~SC01b::
~SC01c::
~SC01d::
~SC01e::
~SC01f::
~SC020::
~SC021::
~SC022::
~SC023::
~SC024::
~SC025::
~SC026::
~SC027::
~SC028::
~SC029::
~SC02a::
~SC02b::
~SC02c::
~SC02d::
~SC02e::
~SC02f::
~SC030::
~SC031::
~SC032::
~SC033::
~SC034::
~SC035::
~SC036::
~SC037::
~SC038::
~SC03a::
~SC03b::
~SC03c::
~SC03d::
~SC03e::
~SC03f::
~SC040::
~SC041::
~SC042::
~SC043::
~SC044::
~SC045::
~SC046::
~SC047::
~SC048::
~SC049::
~SC04a::
~SC04b::
~SC04c::
~SC04d::
~SC04e::
~SC04f::
~SC050::
~SC051::
~SC052::
~SC053::
~SC054::
~SC055::
~SC056::
~SC057::
~SC058::
~SC059::
~SC05a::
~SC05b::
~SC05c::
~SC05d::
~SC05e::
~SC05f::
~SC060::
~SC061::
~SC062::
~SC063::
~SC064::
~SC065::
~SC066::
~SC067::
~SC068::
~SC069::
~SC06a::
~SC06b::
~SC06c::
~SC06d::
~SC06e::
~SC06f::
~SC070::
~SC071::
~SC072::
~SC073::
~SC074::
~SC075::
~SC076::
~SC077::
~SC078::
~SC079::
~SC07a::
~SC07b::
~SC07c::
~SC07d::
~SC07e::
~SC07f::
~SC080::
~SC081::
~SC082::
~SC083::
~SC084::
~SC085::
~SC086::
~SC087::
~SC088::
~SC089::
~SC08a::
~SC08b::
~SC08c::
~SC08d::
~SC08e::
~SC08f::
~SC090::
~SC091::
~SC092::
~SC093::
~SC094::
~SC095::
~SC096::
~SC097::
~SC098::
~SC099::
~SC09a::
~SC09b::
~SC09c::
~SC09d::
~SC09e::
~SC09f::
~SC0a0::
~SC0a1::
~SC0a2::
~SC0a3::
~SC0a4::
~SC0a5::
~SC0a6::
~SC0a7::
~SC0a8::
~SC0a9::
~SC0aa::
~SC0ab::
~SC0ac::
~SC0ad::
~SC0ae::
~SC0af::
~SC0b0::
~SC0b1::
~SC0b2::
~SC0b3::
~SC0b4::
~SC0b5::
~SC0b6::
~SC0b7::
~SC0b8::
~SC0b9::
~SC0ba::
~SC0bb::
~SC0bc::
~SC0bd::
~SC0be::
~SC0bf::
~SC0c0::
~SC0c1::
~SC0c2::
~SC0c3::
~SC0c4::
~SC0c5::
~SC0c6::
~SC0c7::
~SC0c8::
~SC0c9::
~SC0ca::
~SC0cb::
~SC0cc::
~SC0cd::
~SC0ce::
~SC0cf::
~SC0d0::
~SC0d1::
~SC0d2::
~SC0d3::
~SC0d4::
~SC0d5::
~SC0d6::
~SC0d7::
~SC0d8::
~SC0d9::
~SC0da::
~SC0db::
~SC0dc::
~SC0dd::
~SC0de::
~SC0df::
~SC0e0::
~SC0e1::
~SC0e2::
~SC0e3::
~SC0e4::
~SC0e5::
~SC0e6::
~SC0e7::
~SC0e8::
~SC0e9::
~SC0ea::
~SC0eb::
~SC0ec::
~SC0ed::
~SC0ee::
~SC0ef::
~SC0f0::
~SC0f1::
~SC0f2::
~SC0f3::
~SC0f4::
~SC0f5::
~SC0f6::
~SC0f7::
~SC0f8::
~SC0f9::
~SC0fa::
~SC0fb::
~SC0fc::
~SC0fd::
~SC0fe::
~SC0ff::
~SC11c:: ; numpadenter
~SC11d:: ; rcontrol
~SC15b:: ; lwin
~SC15c:: ; rwin
~SC15d:: ; appskey
~SC136:: ; rshift
~SC138:: ; ralt
~SC14b:: ; left
~SC148:: ; up
~SC14d:: ; right
~SC150:: ; down
~SC145:: ; numlock
~SC135:: ; numpaddiv
~SC137:: ; printscrn
~SC152:: ; ins
~SC147:: ; home
~SC149:: ; pgup
~SC151:: ; pgdn
~SC14f:: ; end
~SC153:: ; del
	If ( true = blnStats ) {
		intTimeKeys++
		intStatKeys++
	}

; @todo record per-process keystrokes
return