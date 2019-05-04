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
        public static Stopwatch sw;
        public static string lastkey = "";

        public static string keyForEnable = "RButton"; // right mouse button
        public static int waitToNextClick = 400;       // delay between clicks

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

            //start async control method
            Start();

            while (true)
            {
                if (Enabled)
                {
                    //clicker
                    mouse_event(LEFTDOWN_PRESS, Control.MousePosition.X, Control.MousePosition.Y, 0, 0);
                    System.Threading.Thread.Sleep(Clickrate[0]);
                    mouse_event(MLEFTUP_PRESS, Control.MousePosition.X, Control.MousePosition.Y, 0, 0);
                    System.Threading.Thread.Sleep(Clickrate[1]);
                }
            }



        }

        public static async void Start()
        {
            await Task.Run(() => ControlR());

        }

        //async void for control program state
        static async Task ControlR()
        {
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

                        //if key not set (full cycle complete)
                        if (lastkey == "")
                        {
                            //check input key
                            if(key == keyForEnable)

                            //write key to lastkeys
                            lastkey = key;

                            //start timer
                            sw = Stopwatch.StartNew();
                        }
                        else {
                            //stop timer
                            sw.Stop();  
                            
                            //if key equal 
                            if (key == keyForEnable) {

                                //we should click as fast as possible to activate or disable Clicker (<400ms)
                                if (sw.ElapsedMilliseconds <= waitToNextClick)
                                {
                                    //disable or enable

                                    Enabled = !Enabled;
                                    if (Enabled)
                                    {
                                        Console.WriteLine("Clicker enabled");
                                    }
                                    else
                                    {
                                        Console.WriteLine("Clicker disabled");
                                    }

                                }
                                else {
                                    Console.WriteLine("Fail enable : Too slow click..");
                                }

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
    }
}
