# Fixing __FLYFF_INITPAGE_EXT
_There are a few issues with this system.._

- **CNeuzApp::Render()**  
_If the clients active player doesn't exist, then just use the world that is stored in the theme.__
  ```cpp
		void RenderShadowMap( LPDIRECT3DDEVICE9 pd3dDevice, CObj **pList, int nMax );
	#ifndef __FLYFF_INITPAGE_EXT
		if( g_pPlayer )
		{
			CWorld *pWorld = g_pPlayer->GetWorld();
	#else
		{
			CWorld* pWorld = g_pPlayer ? g_pPlayer->GetWorld() : CWndBase::m_Theme.m_pTitleWorld;
	#endif
			if( pWorld )
  ```
	
- **CMover::ProcessQuest()**  
_If we're processing npc's for the "login Map", then the active mover needs to exist or a crash may happen._
	```cpp
		CMover* pMover = GetActiveMover();
	#ifdef __FLYFF_INITPAGE_EXT
		if (pMover)
		{
	#endif
			m_nQuestEmoticonIndex = -1;
			
			...
			
	#ifdef __FLYFF_INITPAGE_EXT
		}
	#endif
	```  

- **CObj::Process()**  
_While rendering objects, if an object supports it, the opacity will be altered due to the camera's distance. In this case, use the login world's camera if the clients active mover object does not exist._
	```cpp
			float fLengthAct = 1.1f;
	#ifndef __FLYFF_INITPAGE_EXT
		if( GetActiveObj() != NULL )
	#endif
		{
			vPosCam = m_pWorld->m_pCamera->m_vPos;
			vPosObj = GetPos() - vPosCam;
	#ifdef __FLYFF_INITPAGE_EXT
			vPosAct = GetActiveObj() ? GetActiveObj()->GetPos() - vPosCam : CWorld::m_pCamera ? CWorld::m_pCamera->m_vLookAt - vPosCam : vPosObj;
	#else
			vPosAct = GetActiveObj()->GetPos() - vPosCam;
	#endif
	```  
- **CWorld::Process()**  
_Making the skybox render and checking the continent data while g_pPlayer doesn't exist._
	```cpp
		if( g_pPlayer )
		{
			ReadWorld( g_pPlayer->GetPos() );
	#ifndef __FLYFF_INITPAGE_EXT
			if( m_bViewWeather )
			{
				m_skyBox.m_pWorld = this;
				m_skyBox.Process();
			}
	#if __VER >= 15 // __BS_CHANGING_ENVIR
			CheckInOutContinent();
	#endif 
	#endif
		}

	#ifdef __FLYFF_INITPAGE_EXT
		if (m_bViewWeather)
		{
			m_skyBox.m_pWorld = this;
			m_skyBox.Process();
		}
	#if __VER >= 15 // __BS_CHANGING_ENVIR
		CheckInOutContinent();
	#endif 
	#endif
	```  
- **World.h / World.cpp**  
_Move IsUseableDyo and IsUseableDyo2 outside of worldserver so the neuz may use those functions_<br/>  
- World3d.cpp  
_Use camera's location if player doesn't exist. if player doesn't exist, use 0,0,0._  
_CWorld::RenderObject_
	```cpp
				if( g_Option.m_nShadow < 2 )	
			{
	#if __VER >= 14 // __BS_FIX_SHADOW_ONOBJECT
				bool bRenderedShadow = false;
	#ifdef __FLYFF_INITPAGE_EXT
				D3DXVECTOR3 kMyPos = g_pPlayer ? g_pPlayer->GetPos() : (m_pCamera ? D3DXVECTOR3(m_pCamera->GetPos().x, GetLandHeight(m_pCamera->GetPos()), m_pCamera->GetPos().z) : D3DXVECTOR3(0,0,0));			//ÁÖÀÎ°øÀº Ç×»ó À¯È¿ÇÏ´Ù°í °¡Á¤ÇÑ´Ù.
	#else
				D3DXVECTOR3 kMyPos = g_pPlayer->GetPos( );			//ÁÖÀÎ°øÀº Ç×»ó À¯È¿ÇÏ´Ù°í °¡Á¤ÇÑ´Ù.
	#endif
	```
	_void RenderShadowMap_  
	```cpp
		#ifdef __FLYFF_INITPAGE_EXT
			fDistLight = 20.0f + (g_pPlayer ? g_pPlayer->m_fDistCamera : (CWorld::m_pCamera ? CWorld::m_pCamera->GetPos().y : 0)) - 4.0f * 5.375f;
		#else
			fDistLight = 20.0f + (g_pPlayer->m_fDistCamera - 4.0f) * 5.375f;
		#endif

			D3DXVECTOR3 vLightPos = vLightDir * -fDistLight;		// ºû¹æÇâÀ¸·Î 28m ¶³¾îÁø°÷¿¡¼­ ºûÀÌ ºñÃßµµ·Ï ÇÑ´Ù. 28ÀÌ ÀûÁ¤°ª.
		#ifdef __FLYFF_INITPAGE_EXT
			D3DXVECTOR3 v1 = g_pPlayer ? g_pPlayer->GetPos() : CWorld::m_pCamera ? CWorld::m_pCamera->m_vLookAt : D3DXVECTOR3(0, 0, 0);
		#else
			D3DXVECTOR3 v1 = g_pPlayer->GetPos();
		#endif
			D3DXVECTOR3 v2 = CWorld::m_pCamera->m_vPos;
	```  
	_CWorld::SetLight_  
	_fixing light by using cameras lookat vector_
	```cpp
	#if __VER >= 15 // __BS_CHANGING_ENVIR
	#ifdef __FLYFF_INITPAGE_EXT
		D3DXVECTOR3 vecTemp = g_pPlayer ? g_pPlayer->GetPos() : (m_pCamera ? m_pCamera->m_vLookAt : D3DXVECTOR3(0, 0, 0));
		ENVIR_INFO* pInfo = GetInContinent(vecTemp);
	#else
		ENVIR_INFO* pInfo = GetInContinent(g_pPlayer->GetPos());
	#endif
		if( pInfo && m_kCurContinent._bUseEnvir )		// ´ë·ú ¾ÈÀÌ°í ´ë·úÁ¤º¸¸¦ ÀÌ¿ëÇÒ °æ¿ì¸¸ !!
	```
- WorldEnvironment.cpp  
**CWorld::InitAfterCreatedPlayer**  
	```cpp
	#ifdef __FLYFF_INITPAGE_EXT
		ENVIR_INFO* pInfo = GetInContinent(g_pPlayer ? g_pPlayer->GetPos() : m_pCamera ? m_pCamera->GetPos() : D3DXVECTOR3(0,0,0));
	#else
		ENVIR_INFO* pInfo = GetInContinent( g_pPlayer->GetPos() );
	#endif
	```
	**CWorld::CheckInOutContinent**
	```cpp
	#ifdef __FLYFF_INITPAGE_EXT
		ENVIR_INFO* pInfo = GetInContinent(g_pPlayer ? g_pPlayer->GetPos() : m_pCamera ? m_pCamera->GetPos() : D3DXVECTOR3(0,0,0));
	#else
		ENVIR_INFO* pInfo = GetInContinent( g_pPlayer->GetPos( ) );
	#endif
	```
- ITheme.cpp
	```cpp	
	#ifdef __FLYFF_INITPAGE_EXT
	void CTheme::ReadTitleWorld()
	{
		if(m_pTitleWorld == NULL)
		{
			if (g_Option.m_nShadow < 2)
				const bool bSuccess = CreateShadowMap(m_pd3dDevice, D3DFMT_R5G6B5);
			m_pTitleWorld = new CWorld;
			if(m_pTitleWorld != NULL)
			{
				if(m_pTitleWorld->InitDeviceObjects( m_pd3dDevice ) == S_OK)
				{
					if(m_pTitleWorld->OpenWorld( MakePath( DIR_WORLD, "WdMadrigal" ), TRUE ))
					{
						const D3DXVECTOR3 vecPos(6955, 115, 3265);
						m_pTitleWorld->ReadWorld(vecPos);
						m_pTitleWorld->loadDyo();
					}
					else
						SAFE_DELETE(m_pTitleWorld);
				}				
			}
		}
	}

	void CTheme::DestoryTitleWorld()
	{
		if(m_pTitleWorld != NULL)
		{
			m_pTitleWorld->InvalidateDeviceObjects();
			m_pTitleWorld->DeleteDeviceObjects();
			SAFE_DELETE(m_pTitleWorld);
		}

		m_dwTexturAlpha1 = 0;
		m_dwTexturAlpha2 = 0;
	}
	#endif //__FLYFF_INITPAGE_EXT
	HRESULT CTheme::FrameMove()
	{
	#ifdef __FLYFF_INITPAGE_EXT
		if(m_pTitleWorld != nullptr)
		{
			D3DXVECTOR3 vecPos(6955, 115, 3265);
			const D3DXVECTOR3 vecLookat(6925, 105, 3101);

			CCamera camera;
			camera.SetPos(vecPos);
			camera.m_vLookAt = vecLookat;
			m_pTitleWorld->SetCamera(&camera);
			m_pTitleWorld->Process();
		}
	#endif //__FLYFF_INITPAGE_EXT
		return S_OK;
	}	
	void CTheme::RenderDesktop( C2DRender* p2DRender )
	{
	#ifdef __FLYFF_INITPAGE_EXT
		D3DVIEWPORT9 viewport;
		viewport.Width = 1360;
		viewport.Height = 768;

		const float fAspect = static_cast<float>(viewport.Width) / static_cast<float>(viewport.Height);
		constexpr float fFov = D3DX_PI / 4.0f;
		const float fNear = CWorld::m_fNearPlane;

		D3DXMatrixPerspectiveFovLH(&CWorld::m_matProj, fFov, fAspect, fNear - 0.01f, CWorld::m_fFarPlane);
		p2DRender->m_pd3dDevice->SetTransform(D3DTS_PROJECTION, &CWorld::m_matProj);

		const unsigned long dwColor = CWorld::GetDiffuseColor();
		p2DRender->m_pd3dDevice->Clear(0, nullptr, D3DCLEAR_ZBUFFER | D3DCLEAR_TARGET, dwColor, 1.0f, 0);

		if (m_pTitleWorld != nullptr)
		{
			D3DXVECTOR3 vecPos(6955, 115, 3265);
			const D3DXVECTOR3 vecLookat(6925, 105, 3101);

			CCamera camera;
			camera.SetPos(vecPos);
			camera.m_vLookAt = vecLookat;
			m_pTitleWorld->SetCamera(&camera);	
			m_pTitleWorld->Render(p2DRender->m_pd3dDevice, m_pFontWorld);

			p2DRender->m_pd3dDevice->SetRenderState(D3DRS_ALPHABLENDENABLE, TRUE);
			p2DRender->m_pd3dDevice->SetRenderState(D3DRS_SRCBLEND, D3DBLEND_SRCALPHA);
			p2DRender->m_pd3dDevice->SetRenderState(D3DRS_DESTBLEND, D3DBLEND_INVSRCALPHA);
			p2DRender->m_pd3dDevice->SetRenderState(D3DRS_ZWRITEENABLE, FALSE);
			p2DRender->m_pd3dDevice->SetRenderState(D3DRS_ZENABLE, FALSE);
			p2DRender->m_pd3dDevice->SetRenderState(D3DRS_ALPHABLENDENABLE, TRUE);
			p2DRender->m_pd3dDevice->SetRenderState(D3DRS_ALPHATESTENABLE, TRUE);
			p2DRender->m_pd3dDevice->SetRenderState(D3DRS_ALPHAREF, 0x08);
		}
	#else 
		CTexture texture = m_texWallPaper;	
		texture.SetAutoFree( FALSE );
		p2DRender->m_pd3dDevice->SetRenderState( D3DRS_ZWRITEENABLE, FALSE );
		CRect rectWindow = p2DRender->m_clipRect;
		if( m_dwWallPaperType == WPT_STRETCH ) // ÀüÃ¼ ´Ã¸®±â 
		{
			texture.m_size.cx = rectWindow.Width();
			texture.m_size.cy = rectWindow.Height();
			p2DRender->m_pd3dDevice->Clear(0, NULL,  D3DCLEAR_TARGET, m_d3dcBackground, 1.0f, 0 ) ;
			p2DRender->RenderTexture( CPoint( 0, 0 ), &texture );
		}
		else if( m_dwWallPaperType == WPT_CENTER ) // Áß¾Ó Á¤·Ä 
		{
			CPoint pt( ( rectWindow.Width() / 2 ) - ( texture.m_size.cx / 2 ), ( rectWindow.Height() / 2 ) - ( texture.m_size.cy / 2 ) );
			p2DRender->m_pd3dDevice->Clear(0, NULL,  D3DCLEAR_TARGET, m_d3dcBackground, 1.0f, 0 ) ;
			p2DRender->RenderTexture( pt, &texture );
		}
		else if( m_dwWallPaperType == WPT_CENTERSTRETCH ) // Áß¾Ó ´Ã¸®±â  
		{
			if(( (int) rectWindow.Width() - texture.m_size.cx ) < ( (int)rectWindow.Height() - texture.m_size.cy ) )
			{
				texture.m_size.cy = rectWindow.Width() * texture.m_size.cy / texture.m_size.cx;
				texture.m_size.cx = rectWindow.Width();
			}
			else
			{
				texture.m_size.cx = rectWindow.Height() * texture.m_size.cx / texture.m_size.cy;
				texture.m_size.cy = rectWindow.Height();
			}

			CPoint pt( ( rectWindow.Width() / 2 ) - ( texture.m_size.cx / 2 ), ( rectWindow.Height() / 2 ) - ( texture.m_size.cy / 2 ) );
			p2DRender->m_pd3dDevice->Clear(0, NULL,  D3DCLEAR_TARGET, m_d3dcBackground, 1.0f, 0 ) ;
			p2DRender->RenderTexture( pt, &texture );
		}
		else if( m_dwWallPaperType == WPT_TILE ) // Å¸ÀÏ Á¤·Ä 
		{
			FLOAT fu = (FLOAT)rectWindow.Width()  / texture.m_size.cx;
			FLOAT fv = (FLOAT)rectWindow.Height() / texture.m_size.cy;
			texture.m_size.cx = rectWindow.Width();
			texture.m_size.cy = rectWindow.Height();
			texture.m_fuLT = 0.0f; texture.m_fvLT = 0.0f;
			texture.m_fuRT = fu  ; texture.m_fvRT = 0.0f;
			texture.m_fuLB = 0.0f; texture.m_fvLB = fv  ;
			texture.m_fuRB = fu  ; texture.m_fvRB = fv  ;
			p2DRender->m_pd3dDevice->Clear(0, NULL,  D3DCLEAR_TARGET, m_d3dcBackground, 1.0f, 0 ) ;
			p2DRender->RenderTexture( CPoint( 0, 0), &texture );
		}
		p2DRender->m_pd3dDevice->SetRenderState( D3DRS_ALPHABLENDENABLE, TRUE );
		p2DRender->m_pd3dDevice->SetRenderState( D3DRS_CULLMODE, D3DCULL_NONE );
	#endif 
	}
	```

