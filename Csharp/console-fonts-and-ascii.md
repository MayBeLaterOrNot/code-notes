# Console Application Fonts and Extended ASCII

By default it is difficult to render non-standard (extended ASCII and/or unicode) characters. If the console itself doesn't support the encoding, the default font often doesn't support the characters. Registry hacks can change the default console font system-wide, but it requires a restart and doesn't work with Visual Studio's console (launched with F5).

The best way I found to render things like ASCII line art is to create a class to interact with kernel32.dll. This requires enabling unsafe code though (project -> project properties -> build -> allow unsafe code). 

Monospace fonts which support ASCII line art are `Consolas` and `Courier New`.

## ConsoleStyler.cs
```cs
public class ConsoleStyler
{
    public ConsoleStyler()
    {
    }
    
    public void SetConsoleFont(string fontName = "Arial") // Lucida Console
    {
        unsafe
        {
            IntPtr hnd = GetStdHandle(STD_OUTPUT_HANDLE);
            if (hnd != INVALID_HANDLE_VALUE)
            {
                CONSOLE_FONT_INFO_EX info = new CONSOLE_FONT_INFO_EX();
                info.cbSize = (uint)Marshal.SizeOf(info);

                // Set console font
                CONSOLE_FONT_INFO_EX newInfo = new CONSOLE_FONT_INFO_EX();
                newInfo.cbSize = (uint)Marshal.SizeOf(newInfo);
                newInfo.FontFamily = TMPF_TRUETYPE;
                IntPtr ptr = new IntPtr(newInfo.FaceName);
                Marshal.Copy(fontName.ToCharArray(), 0, ptr, fontName.Length);

                // Get some settings from current font
                double sizeY = 16;
                double sizeX = sizeY;
                newInfo.dwFontSize = new COORD((short)(sizeX), (short)(sizeY));
                newInfo.FontWeight = info.FontWeight;
                SetCurrentConsoleFontEx(hnd, false, ref newInfo);
            }
        }
    }

    [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode)]
    internal unsafe struct CONSOLE_FONT_INFO_EX
    {
        internal uint cbSize;
        internal uint nFont;
        internal COORD dwFontSize;
        internal int FontFamily;
        internal int FontWeight;
        internal fixed char FaceName[LF_FACESIZE];
    }

    [StructLayout(LayoutKind.Sequential)]
    internal struct COORD
    {
        internal short X;
        internal short Y;

        internal COORD(short x, short y)
        {
            X = x;
            Y = y;
        }
    }

    private const int STD_OUTPUT_HANDLE = -11;
    private const int TMPF_TRUETYPE = 4;
    private const int LF_FACESIZE = 32;
    private static IntPtr INVALID_HANDLE_VALUE = new IntPtr(-1);

    [DllImport("kernel32.dll", SetLastError = true)]
    static extern bool SetCurrentConsoleFontEx(
        IntPtr consoleOutput,
        bool maximumWindow,
        ref CONSOLE_FONT_INFO_EX consoleCurrentFontEx);

    [DllImport("kernel32.dll", SetLastError = true)]
    static extern IntPtr GetStdHandle(int dwType);

    [DllImport("kernel32.dll", SetLastError = true)]
    static extern int SetConsoleFont(
        IntPtr hOut,
        uint dwFontNum
        );
}
```