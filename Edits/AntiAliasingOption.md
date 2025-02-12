## MSAA
---
This is a simple implementation of MSAA. It has been around for a while, but I wanted to show that you are able to downscale to a supported type rather than giving the user an error and setting MSAA off.

- CD3DApplication::Initialize3DEnvironment

```cpp

#ifdef __Antialiasing 
	unsigned long MSQuality = 0;
	D3DMULTISAMPLE_TYPE optLvl = D3DMULTISAMPLE_NONE;
	if (g_Option.MultiSampling > 4)
		g_Option.MultiSampling = 4;

	while (g_Option.MultiSampling > 0)
	{
		optLvl = static_cast<D3DMULTISAMPLE_TYPE>(std::pow(2, g_Option.MultiSampling));
		if (SUCCEEDED(m_pD3D->CheckDeviceMultiSampleType(D3DADAPTER_DEFAULT, D3DDEVTYPE_HAL, D3DFMT_A8R8G8B8, TRUE, optLvl, &MSQuality)))
		{
			m_d3dpp.MultiSampleType = optLvl;
			m_d3dpp.MultiSampleQuality = MSQuality - 1;
			break;
		}
		--g_Option.MultiSampling;
	}
	m_d3dpp.SwapEffect = D3DSWAPEFFECT_DISCARD;
	m_d3dpp.BackBufferFormat = D3DFMT_A8R8G8B8;
	m_d3dpp.EnableAutoDepthStencil = TRUE;
	m_d3dpp.AutoDepthStencilFormat = D3DFMT_D16;
	m_d3dpp.Flags = 0;
#endif

	hr = m_pD3D->CreateDevice( m_d3dSettings.AdapterOrdinal(), pDeviceInfo->DevType,
								m_hWndFocus, behaviorFlags, &m_d3dpp,
								&m_pd3dDevice );

#ifdef __Antialiasing 
		if (g_Option.MultiSampling)
			m_pd3dDevice->SetRenderState(D3DRS_MULTISAMPLEANTIALIAS, TRUE);
#endif
```
- HwOption.cpp
```cpp
		else if (scan.Token == _T("Multisampling")) // __AntiAliasing
			MultiSampling = scan.GetNumber();
```
```cpp
	_ftprintf(fp, _T("Multisampling %d\n"), MultiSampling);
```
- hwoption.h
```cpp
	unsigned char MultiSampling{3};
```

> Note: If you want to store it as : 0, 2, 4, 8, 16, instead of 0-4, you have to change the std::pow call.