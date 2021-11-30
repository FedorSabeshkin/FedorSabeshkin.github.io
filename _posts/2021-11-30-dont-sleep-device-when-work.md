---
layout: post
title:  "Как предотвратить засыпание устройства во время выполнения долгой задачи, но сохранить энергосберегающий режим работы"
feature_img: /assets/images/2021/November/2021-11-30-dont-sleep-device-when-work/date_and_time_wince.gif
---

Уточнение: я буду рассматривать пример устройства на WinCE, на Andorid и iOS может быть иная обработка.

Во время разработки мобильного приложения, время на загрузку приложением данных  - может превышать время через которое устройство переходит в спящий режим - тогда ОС девайса может закрыть все подключения, не дожидаясь окончания получения информации. 

Как быть в такой ситуации?

1. Вручную в настройках системы увеличить время бездействия, после которого система переходит в спящий режим. 

Минусы: 
- Как быть, если имеется большой парк устройств и быть уверенным, что все пользователи проставили эту настройку? 

2. Автоматически, из нашей программы, изменить время после которого система переходит в спящий режим, так, что бы загрузка успевала пройти до его истечения. 

Минусы: 
- Нет гарантии, что загрузка успеет завершится даже в увеличенный промежуток времени, если интернет соединение будем медленее, или объем данных окажется больше ожидаемого.
- Батарея будет садиться быстрее, чем до установки вашего приложения, т.к. теперь устройство при бездействии позже уходит в спящий режим. Пользоваться им становится менее приятно.

3. Не давать девайсу засыпать, именно тогда, когда наша программа еще выполняет на нем работу. В остальных случаях должны работать ранее установленные настройки системы.

Механизм решения.
В WinCE мы можем получить доступ до значения таймера, которое используется ОС при проверке о том, что пора переходить в спящий режим. 
Обновляя это значение на его начальное, раньше чем пройдет заданное им время - мы сможем предотвратить переход системы в спящий режим. 
Когда же наш процесс завершится - мы может просто отключить это обновление значения и устройство заснет, так как ему следует относительно пользовательской настройки.


Для сброса таймера в приложениях на Compact Framework можно использовать функцию [SystemIdleTimerReset](https://www.pinvoke.net/default.aspx/coredll/SystemIdleTimerReset.html) вызываемую через P/Invoke. 
Что бы время контроллировалось паралельно выполнению нашей основной программы, будем создавать отдельный тред.
Код ниже объявляет две функции, которые Вы можете вызывать в начале загрузки файла, что бы запустить сбрасывание таймер через n-ый промежуток времени и прекращение этого, когда продолжительная операция закончена (скачан файл или получена ошибка).

{% highlight C# %}


    /// <summary>
    /// This function resets a system timer that controls whether or not the
    /// device will automatically go into a suspended state.
    /// </summary>
    [DllImport("CoreDll.dll")]
    public static extern void SystemIdleTimerReset();

    private static int nDisableSleepCalls = 0;
    private static System.Threading.Timer preventSleepTimer = null;

    public static void DisableDeviceSleep(int seconds)
    {
        nDisableSleepCalls++;
        if (nDisableSleepCalls == 1)
        {
            Debug.Assert(preventSleepTimer == null);
            int ms = Constants.IdleTimerMs;
            // start a periodic timer
            preventSleepTimer = new System.Threading.Timer(new TimerCallback(PokeDeviceToKeepAwake),
                null, 0, ms);
        }
    }

    public static void EnableDeviceSleep()
    {
        nDisableSleepCalls--;
        if (nDisableSleepCalls == 0)
        {
            Debug.Assert(preventSleepTimer != null);
            if (preventSleepTimer != null)
            {
                preventSleepTimer.Dispose();
                preventSleepTimer = null;
            }
        }
    }

    private static void PokeDeviceToKeepAwake(object extra)
    {
        try
        {
            SystemIdleTimerReset();
        }
        catch (Exception exception)
        {
            Logger.Log(exception.ToString());
        }
    }

    // from Constants.cs file
    public const int IdleTimerMs = 30;

{% endhighlight %}

Соответсвенно в своем методе выполняющем продолжительную работу, вам достаточно добавить вызовы DisableDeviceSleep() в начале и EnableDeviceSleep() в конце.


{% highlight C# %}
       public void LoadFile()
        {
            DisableDeviceSleep();
            // Load file
            EnableDeviceSleep()
        }

{% endhighlight %}

Не забудьте обработать случаи, для возврата стандартного поведения таймера засыпания - вызов EnableDeviceSleep(), если выполение основного метода неожиданно прервется.