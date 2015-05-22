title: Python GUI
date: 2015-04-01 14:53:29
tags: Python
---

1.PythonでGUI画面をつくろうとするとき、wxPythonを使います。だけど、最初は[ImportError: DLL load failed: %1 は有効な Win32 アプリケーションではありません]
問題が出ます。
僕が以下の解決方法を使って、解決しました。
このエラーの原因の1つは：PythonバージョンとwxPythonのバージョンが合わないです。僕のパソコンでインストールしたPythonバージョンはPython2.7 32bit windowsだけど、
インストールしたwxPythonのバージョンはPython3.0 64bit