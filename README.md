# Simple C# clicker

This project contains a simple clicker code. Use it for yourself.
Code is here:

```csharp
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace simple_clicker
{
    class Program
    {
        [System.Runtime.InteropServices.DllImport("user32.dll")]
        static extern bool SetCursorPos(int x, int y);

        [System.Runtime.InteropServices.DllImport("user32.dll")]
        public static extern void mouse_event(int dwFlags, int dx, int dy, int cButtons, int dwExtraInfo);

        [DllImport("user32.dll")]
        public static extern int GetAsyncKeyState(Int32 i);

        public const int LEFTDOWN_PRESS = 0x02;
        public const int MLEFTUP_PRESS = 0x04;

        //for enable clicker
        public static string lastkey;
        public static Stopwatch sw = Stopwatch.StartNew();
        public static string keyForEnable = "RButton"; // right mouse button
        public static int waitToNextClick = 450;       // delay between clicks

        //is enabled clicker
        public static bool Enabled = false;


        public static int[] Clickrate = {
                           10,      //how long keep pressed
                           100      //how long sleep to next click
  };

        static void Main(string[] args)
        {
            Console.WriteLine("---------\n\n" +
                "Simple clicker on C#" +
                "\n\nTo enable clicker:" +
                "\n   Double press Right Mouse Button" +
                "\n   Same for disable clicker" +
                "\n\n---------");

            //start async clicker
            StartClicker();


            while (true)
            {
                Thread.Sleep(10);
                for (Int32 i = 0; i < 255; i++)
                {
                    int keyState = GetAsyncKeyState(i);
                    if (keyState == 1 || keyState == -32767)
                    {
                        //whats key is pressed
                        string key = ((Keys)i).ToString();

                        // we need RButton only
                        if (key == keyForEnable) {

                            if (lastkey == "")
                            {
                                //start to new cricle
                                sw.Reset();
                                sw.Start();
                                lastkey = "1";
                            }
                            else {

                                //check the time
                                sw.Stop();

                                //you should click as fast as possible to activate or disable Clicker (<450ms)
                                if (sw.ElapsedMilliseconds <= waitToNextClick)
                                {
                                    Enabled = !Enabled; // invert
                                    Console.WriteLine("Change status :: {0}", Enabled);
                                }
                                    
                                else
                                    Console.WriteLine("Too slow!");

                                //reset all
                                lastkey = "";
                                sw.Reset();
                            }

                        }

                        break;
                    }
                }
            }




        }

        public static async void StartClicker()
        {
            await Task.Run(() => Clicker());

        }

        //async void for control program state
        static async Task Clicker()
        {
            while (true)
            {
                if (Enabled)
                {
                    //clicker
                    mouse_event(LEFTDOWN_PRESS, Control.MousePosition.X, Control.MousePosition.Y, 0, 0);
                    Thread.Sleep(Clickrate[0]);
                    mouse_event(MLEFTUP_PRESS, Control.MousePosition.X, Control.MousePosition.Y, 0, 0);
                    Thread.Sleep(Clickrate[1]);
                }
            }
        }
    }
}
