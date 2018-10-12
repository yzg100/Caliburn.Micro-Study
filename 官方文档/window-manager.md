---
layout: page
title: The Window Manager
---

It is quite hard to write documentation for the Window Manager, as there is not one Window Manager but there is one per platform (Silverlight, Phone, WPF and WinRT). To make it even more complex: these Windows Managers have different interfaces and features. 

---
><font color="#63aebb" face="微软雅黑">为窗口管理器编写文档是相当困难的，因为它不是唯一的，每个平台都有一个（Silverlight，Phone，WPF和WinRT）。使其更加复杂：这些窗口管理器具有不同的界面和功能。</font>

### Silverlight

A Silverlight application consists of one root visual only. The Window Manager is used to show dialogs, popups and toast notifications only.

---
><font color="#63aebb" face="微软雅黑">Silverlight应用程序仅由一个根visual元素组成。窗口管理器仅用于显示对话框、弹出窗口和 toast 通知。</font>

### WPF

The Window Manager is used for every Window that is shown (modal and non-modal).

---
><font color="#63aebb" face="微软雅黑">窗口管理器用于显示的每个窗口(模态和非模态)。</font>

### Windows Phone

Windows Phone uses the View-First approach to show the applications pages. The Window Manager is used to show dialogs and popups on top of a page only.

---
><font color="#63aebb" face="微软雅黑">Windows Phone使用View-First方法显示应用程序页面。窗口管理器用于仅在页面顶部显示对话框和弹出窗口。</font>

### WinRT

A Windows Store application consists of multiple pages (similar to Windows Phone). The Window Manager is used to show the Settings Flyout only.

---
><font color="#63aebb" face="微软雅黑">Windows 应用商店应用程序由多个页面组成（类似于Windows Phone）。窗口管理器仅用于显示设置弹出窗口。</font>

[目录](index)&nbsp;&nbsp;|&nbsp;&nbsp;[Design Time Support - 设计时支持](./design-time)