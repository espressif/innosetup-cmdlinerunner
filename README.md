# InnoSetup cmdlinerunner

[cmdlinerunner](cmdlinerunner.c) — a helper DLL used to run external command-line programs from the installer, capture live console output, and get the exit code.

Example project: https://github.com/espressif/idf-installer/tree/main/src/InnoSetup

Example of usage:

```pascal
[Files]
Source: "lib\cmdlinerunner.dll"; Flags: dontcopy

{ ------------------------------ Helper functions from libcmdlinerunner.dll ------------------------------ }
function ProcStart(cmdline, workdir: string): Longword;
  external 'proc_start@files:cmdlinerunner.dll cdecl';
function ProcGetExitCode(inst: Longword): DWORD;
  external 'proc_get_exit_code@files:cmdlinerunner.dll cdecl';
function ProcGetOutput(inst: Longword; dest: PAnsiChar; sz: DWORD): DWORD;
  external 'proc_get_output@files:cmdlinerunner.dll cdecl';
procedure ProcEnd(inst: Longword);
  external 'proc_end@files:cmdlinerunner.dll cdecl';

function ExecuteProcess(Command:String):String;
var
  Buffer: String;
  ExitCode: Integer;
  Handle: Longword;
  LogTextAnsi: AnsiString;
  Res: Integer;
begin
    Buffer := '';
    ExitCode := -1;
    Log('Executing: ' + Command);
    Handle := ProcStart(Command, ExpandConstant('{tmp}'))
    if Handle = 0 then
    begin
      Log('ProcStart failed');
      Result := Buffer;
      Exit;
    end;
    while (ExitCode = -1) and not CmdlineInstallCancel do
    begin
      ExitCode := ProcGetExitCode(Handle);
      SetLength(LogTextAnsi, 4096);
      Res := ProcGetOutput(Handle, LogTextAnsi, 4096)
      if Res > 0 then
      begin
        SetLength(LogTextAnsi, Res);
        Buffer := Buffer + String(LogTextAnsi);
      end;
      Sleep(10);
    end;
    ProcEnd(Handle);
    Result := Buffer;
end;
```

## Build

* Build cmdlinerunner DLL.
  - On Windows, it is possible to build using Visual Studio, with CMake support installed
    ```
    mkdir build
    cd build
    cmake -DCMAKE_BUILD_TYPE=Release ..
    cmake --build .  --config Release
    ```
    This will produce `Release/cmdlinerunner.dll`
  - On Linux/Mac, install mingw-w64 toolchain (`i686-w64-mingw32-gcc`). Then build the DLL using CMake:
    ```
    mkdir -p build
    cd build
    cmake -DCMAKE_TOOLCHAIN_FILE=../toolchain-i686-w64-mingw32.cmake -DCMAKE_BUILD_TYPE=Release ..
    cmake --build .
    ```
    This will produce `cmdlinerunner.dll` in the build directory.
