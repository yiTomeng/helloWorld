title: C++反汇编与逆向分析技术揭秘(3)--第三章
date: 2015-03-05 09:42:29
tags: Read Tech
---

1.VC++6.0にて、起動関数はmainCRTStartup,wmainCRTStartup,WinMainCRTStartup,wWinMainCRTStartup
その中に、ヒップの初期化、多数スレッド環境の初期化、引数や環境グローバル変数内容の取得を行う。最後にmain()関数を呼び出す。
mainCRTStartup:
->GetVersion()
->_heap_init()
->GetCommandLineA()
->_crtGetEnvironmentStringsA()
->_setargv()
->_setenvp()
->_cinit()
->main()

VS2005に起動関数の名前は_tmainCRTStartupに変更した。

2.OllyGDBでプログラムを実行すると、最初のmainCRTStartupのところで止まっている。それから、main()関数の特徴により見つける。main()関数の特徴は3つの引数とのことだ。他の関数は3つの引数ではない。