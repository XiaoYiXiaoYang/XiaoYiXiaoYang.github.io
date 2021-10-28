---
title: 网络化测控技术（数据采集）
date: 2020-01-08 11:34:40
tags: [网络化测控]
categories: [Network]
---

<center>
 DI、DO、AI
</center>

<!--more-->

**数字量输入（DI）**

```c++
在OnInitDialog()中设置定时器，添加WM_TIME消息响应函数OnTimer();

在函数OnTimer中调用API获取当前接口的值，该值保存于数组bDISts中，DI0~DI7分别对应数组0~7的数据，然后刷新；

然后添加WM_PAINT消息响应函数OnPaint，在OnPaint中根据当前接口的值判断位图的显示；

具体代码：
添加成员变量：BYTE bDISts[8];

void CDIDLG::OnTimer(UINT nIDEvent) 
{
	HANDLE hDevice;
	int DeviceLgcID = 0; 
	hDevice = USB2832_CreateDevice(DeviceLgcID); //创建设备
	if(hDevice == INVALID_HANDLE_VALUE)
	{
		MessageBox("Create Device Error...\n");
		KillTimer(1);
	}
	if(!USB2832_GetDeviceDI(hDevice, bDISts)) // 开关量输入
	{
		MessageBox("USB2832_GetDeviceDI...\n");
		KillTimer(1);
	}
	Invalidate(TRUE);
	CDialog::OnTimer(nIDEvent);
} 
void CDIDLG::OnPaint() 
{
	CPaintDC dc(this); // device context for painting

	CBitmap bmp; 
	bmp.LoadBitmap(IDB_BITMAP1); 
	CBitmap bmp2;
	bmp2.LoadBitmap(IDB_BITMAP2);

	if (bDISts[0]==0)//接口DI0
	{
		m_bmp.SetBitmap((HBITMAP)bmp); //绿色位图
	}else
	{
		m_bmp.SetBitmap((HBITMAP)bmp2); //红色位图
	}
	bmp.Detach();
}
```



**数字量输出（DO）**

```c++
添加成员变量：BYTE bDISts[8];

添加按钮的响应函数，该函数控制某一接口的数据输出自己选择（DO0~DO7）；

按钮代码：
void CDODLG::OnT0() 
{
	bDOSts[0]++;//记录当前按钮的状态
	if (bDOSts[0]>1)
	{
		bDOSts[0]=0;
	}
	CWnd* pWnd = GetDlgItem(IDC_T0); 
	if (bDOSts[0]==1)//设置按钮的名称
	{ 
		pWnd->SetWindowText(_T("1"));
	}
	else
	{
		pWnd->SetWindowText(_T("0"));
	}
	HANDLE hDevice;
	int DeviceLgcID = 0; 
	hDevice = USB2832_CreateDevice(DeviceLgcID); /*创建设备*/
	if(hDevice == INVALID_HANDLE_VALUE)
	{
		MessageBox("Create Device Error...\n");
		return;
	}
	if(!USB2832_SetDeviceDO(hDevice, bDOSts)) // 开关量输出
	{
		MessageBox("SetDeviceDO Error...\n");
		return;
	}
}

```



**模拟量输入（AI）**

```c++
文本框控件，“采集”按钮，“停止”按钮，组合框控件，文本框控件分别添加成员变量，分别为控件类型m_cobox和字符串类型m_now；

添加添加WM_TIME消息响应函数OnTimer();在“采集”按钮消息响应函数中添加SetTimer，“停止”按钮消息响应函数中添加KillTimer；


代码实现：
	int n = m_cobox.GetCurSel();
	
	HANDLE hDevice;         // 设备对象句柄
	int DeviceLgcID;        // 物理设备ID号(由板上JP1决定)
	BOOL bReturn;           // 函数的返回值
	int nReadSizeWords;     // 每次读取AD数据个数
	LONG nRetWords;         // 实际读取的数据个数
	int nChannelCount = 0;  // 采样通道数
	WORD ADBuffer[32768];   // 接收AD 数据的缓冲区
	WORD ADData;
	float Volt;             // 将AD原始数据转换为电压值
	int nRemainder = 0;

	USB2832_PARA_AD ADPara;         // 初始化AD的参数结构
	ADPara.FirstChannel		= n;                                  // 首通道0
	ADPara.LastChannel		= n;                                  // 末通道3
	ADPara.InputRange		= 1;                                  // 量程选择
	ADPara.Gains			= USB2832_GAINS_1MULT;	        // 使用1倍增益
	ADPara.GroundingMode	= USB2832_GNDMODE_SE; // 单端方式
	nChannelCount = ADPara.LastChannel - ADPara.FirstChannel + 1; // 采样通道数

	DeviceLgcID = 0; // 设备ID号, 假设系统中只有一个USB2832设备，即DeviceLgcID=0;
	hDevice = USB2832_CreateDevice(DeviceLgcID);				  // 创建设备对象
	if(hDevice == INVALID_HANDLE_VALUE) 
	{ 
		printf("Create Device Error\n"); 
		return; 
	}

	bReturn = USB2832_InitDeviceAD(hDevice, &ADPara); // 初始化AD
	if (!bReturn)
	{
		printf( "USB2832_InitDeviceAD Error\n" );
	}

	nReadSizeWords = 128;// 读取数据的大小
	printf("请等待，您可以按任意键退出，但请不要直接关闭窗口强制退出...\n");
	if(!USB2832_ReadDeviceAD(hDevice, ADBuffer, nReadSizeWords, &nRetWords))
// 读取AD转换数据
	{
		printf("ReadDeviceAD Error...\n");
	}
	
	int nChannel = ADPara.FirstChannel;
	for (int Index=0; Index<4; Index++) // 总共显示64个点的AD数据
	{
		ADData = ADBuffer[Index]&0x1FFF;
		
		Volt = (float)((10000.00/8192) * ADData - 5000.00); // 将AD数据转换为电压值
		if (nChannel==n)
		{
			m_now.Format("%8.2f",Volt);
			UpdateData(FALSE);
		}

		nChannel++;           // 通道号递加，准备换算下一个通道的数据
		if (nChannel>ADPara.LastChannel) // 如果换算到末通道，再回到首通道
		{
			printf("\n");              // 将显示光标位置移到下一项
			nChannel = ADPara.FirstChannel;
		}
	}                                      // 多点数据换算显示

	USB2832_ReleaseDeviceAD(hDevice); // 释放AD，停止AD数据转换
	USB2832_ReleaseDevice(hDevice);   // 释放设备对象	


	CDialog::OnTimer(nIDEvent);

```





2019-10-22 21:54:04 星期二