## Anisotropic Option
---

Example code.
```cpp 
#ifdef __Anisotropy 
	if (g_Option.Anisotropic > 0)
	{
		pd3dDevice->SetSamplerState(0, D3DSAMP_MINFILTER, D3DTEXF_ANISOTROPIC);
		pd3dDevice->SetSamplerState(0, D3DSAMP_MAGFILTER, D3DTEXF_ANISOTROPIC);
		pd3dDevice->SetSamplerState(0, D3DSAMP_MIPFILTER, D3DTEXF_ANISOTROPIC);
		pd3dDevice->SetSamplerState(0, D3DSAMP_MAXANISOTROPY, g_Option.Anisotropic);
	}
	else
#endif 
	{
		pd3dDevice->SetSamplerState(0, D3DSAMP_MINFILTER, D3DTEXF_LINEAR);
		pd3dDevice->SetSamplerState(0, D3DSAMP_MAGFILTER, D3DTEXF_LINEAR);
	}
```

make sure to check caps in Initialize3dEnvironment in d3dapp
```cpp
        m_pd3dDevice->GetDeviceCaps( &m_d3dCaps );
        m_dwCreateFlags = behaviorFlags;

		if (m_d3dCaps.MaxAnisotropy < g_Option.Anisotropic)
			g_Option.Anisotropic = m_d3dCaps.MaxAnisotropy;
```

