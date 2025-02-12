## Camera Options
---
The game is missing options for same camera related things, like rotation speed, disabling roll-ing scroll, and inversion.

- HwOption.h
```cpp
	bool Camera_InvertScroll = false;
	bool Camera_AngleScroll = true;
	float Camera_ZoomSpeed = 0.5f;
	float Camera_RotateSpeed_Mouse = 0.2f;
	float Camera_RotateSpeed_Keys = 4.0f;
	bool Camera_RotateInvert_Mouse = false;
	bool Camera_RotateInvert_KeysXZ = false;
	bool Camera_RotateInvert_KeysY = false;
```

- HwOption.cpp
```cpp
		else if (scan.Token == _T("Camera_AngleScroll"))
			Camera_AngleScroll = scan.GetNumber() != 0;
		else if (scan.Token == _T("Camera_InvertScroll"))
			Camera_InvertScroll = scan.GetNumber() != 0;
		else if (scan.Token == _T("Camera_ZoomSpeed"))
			Camera_ZoomSpeed = scan.GetFloat();
		else if (scan.Token == _T("Camera_RotateSpeed_Mouse"))
			Camera_RotateSpeed_Mouse = scan.GetFloat();
		else if (scan.Token == _T("Camera_RotateSpeed_Keys"))
			Camera_RotateSpeed_Keys = scan.GetFloat();
		else if (scan.Token == _T("Camera_RotateInvert_Mouse"))
			Camera_RotateInvert_Mouse = scan.GetNumber() != 0;
		else if (scan.Token == _T("Camera_RotateInvert_KeysXZ"))
			Camera_RotateInvert_KeysXZ = scan.GetNumber() != 0;
		else if (scan.Token == _T("Camera_RotateInvert_KeysY"))
			Camera_RotateInvert_KeysY = scan.GetNumber() != 0;

```
```cpp
	_ftprintf(fp, _T("Camera_InvertScroll %d\n"), static_cast<int>(Camera_InvertScroll));
	_ftprintf(fp, _T("Camera_AngleScroll %d\n"), static_cast<int>(Camera_AngleScroll));
	_ftprintf(fp, _T("Camera_ZoomSpeed %.1f\n"), Camera_ZoomSpeed);
	_ftprintf(fp, _T("Camera_RotateSpeed_Mouse %.1f\n"), Camera_RotateSpeed_Mouse);
	_ftprintf(fp, _T("Camera_RotateSpeed_Keys %.1f\n"), Camera_RotateSpeed_Keys);
	_ftprintf(fp, _T("Camera_RotateInvert_Mouse %d\n"), static_cast<int>(Camera_RotateInvert_Mouse));
	_ftprintf(fp, _T("Camera_RotateInvert_KeysXZ %d\n"), static_cast<int>(Camera_RotateInvert_KeysXZ));
	_ftprintf(fp, _T("Camera_RotateInvert_KeysY %d\n"), static_cast<int>(Camera_RotateInvert_KeysY));
```

- WndWorld.cpp
<small>*CWndWorld::OnMouseWheel*</small>
```cpp
	#ifdef __CameraOptions
		if ((g_Option.Camera_InvertScroll ? zDelta < 0 : zDelta > 0))
	#else
		if (zDelta < 0)
	#endif
		{
	#ifdef __CameraOptions
			g_Neuz.m_camera.m_fZoom -= g_Option.Camera_ZoomSpeed;
			if (g_Neuz.m_camera.m_fZoom < -1.3f) // lowers min zoom
				g_Neuz.m_camera.m_fZoom = -1.3f;
	#else
			g_Neuz.m_camera.m_fZoom -= 0.5f;

			if (g_Neuz.m_camera.m_fZoom < -1.5f)
				g_Neuz.m_camera.m_fZoom = -1.5f;
	#endif
			if(g_Neuz.m_camera.m_fZoom < -1.5f )
			g_Neuz.m_camera.m_fZoom = -1.5f;
		}
		else
		{
	#ifdef __CameraOptions
			g_Neuz.m_camera.m_fZoom += g_Option.Camera_ZoomSpeed;
	#else
			g_Neuz.m_camera.m_fZoom += 0.5f;
	#endif
```
<small>*CWndWorld::OnMouseMove*</small>
```cpp

#ifdef __CameraOptions
		const float fRotSpeed = g_Option.Camera_RotateSpeed_Mouse;
#else
		FLOAT fRotSpeed = 1.0f;
		switch (g_Option.m_MouseSpeed) // This is normally unused or force set at 1.0f(0)
		{
			case 0:
				fRotSpeed = 1.0f; break;
			case 1:
				fRotSpeed = 0.2f; break;
			case 2:
				fRotSpeed = 0.1f; break;
			default:
				fRotSpeed = 0.2f; break;
		}
#endif		
#ifdef __CameraOptions
		if (g_Option.Camera_RotateInvert_Mouse)
			g_Neuz.m_camera.m_fRotx -= ((point.x - m_ptMouseOld.x) * fRotSpeed);
		else
#endif		
			g_Neuz.m_camera.m_fRotx += ((point.x - m_ptMouseOld.x) * fRotSpeed );

//...emited code to remove bulk...

#ifdef __CameraOptions
		if (g_Option.Camera_RotateInvert_Mouse)
			g_Neuz.m_camera.m_fRoty -= ((point.y - m_ptMouseOld.y) * fRotSpeed);
		else
#endif
			g_Neuz.m_camera.m_fRoty += ((point.y - m_ptMouseOld.y) * fRotSpeed);

#ifdef __CameraOptions
		if (g_Option.Camera_AngleScroll)
		{
			if (g_Neuz.m_camera.m_fRoty > 80 - g_Neuz.m_camera.m_fZoom * 4)
				g_Neuz.m_camera.m_fRoty = 80 - g_Neuz.m_camera.m_fZoom * 4;
		}
		else if (g_Neuz.m_camera.m_fRoty > 80)
			g_Neuz.m_camera.m_fRoty = 80;
#else
		if (g_Neuz.m_camera.m_fRoty > 80 - g_Neuz.m_camera.m_fZoom * 4)
			g_Neuz.m_camera.m_fRoty = 80 - g_Neuz.m_camera.m_fZoom * 4;
#endif
		if(g_Neuz.m_camera.m_fRoty<-80) 
			g_Neuz.m_camera.m_fRoty=-80;
```
<small>*CWndWorld::Process*</small>
```cpp
#ifdef __CameraOptions
			if (g_Option.Camera_RotateInvert_KeysXZ)
				g_Neuz.m_camera.m_fRotx += g_Option.Camera_RotateSpeed_Keys;
			else
				g_Neuz.m_camera.m_fRotx -= g_Option.Camera_RotateSpeed_Keys;
#else
			g_Neuz.m_camera.m_fRotx -= 4;
#endif


//...Im going to emitcode, but you should know what to do from the edits...


#ifdef __CameraOptions
			if (g_Option.Camera_RotateInvert_KeysXZ)
				g_Neuz.m_camera.m_fRotx -= g_Option.Camera_RotateSpeed_Keys;
			else
				g_Neuz.m_camera.m_fRotx += g_Option.Camera_RotateSpeed_Keys;
#else
			g_Neuz.m_camera.m_fRotx += 4;
#endif

// some code emitted

#ifdef __CameraOptions
			if (g_Option.Camera_RotateInvert_KeysY)
				g_Neuz.m_camera.m_fRoty -= 2;
			else
				g_Neuz.m_camera.m_fRoty += 2;

			if (g_Option.Camera_AngleScroll)
			{
				if (g_Neuz.m_camera.m_fRoty > 80 - g_Neuz.m_camera.m_fZoom * 4)
					g_Neuz.m_camera.m_fRoty = 80 - g_Neuz.m_camera.m_fZoom * 4;
			}
#else
			g_Neuz.m_camera.m_fRoty += 2;
			if (g_Neuz.m_camera.m_fRoty > 80 - g_Neuz.m_camera.m_fZoom * 4)
				g_Neuz.m_camera.m_fRoty = 80 - g_Neuz.m_camera.m_fZoom * 4;

#endif

// same again

#ifdef __CameraOptions
			if (g_Option.Camera_RotateInvert_KeysY)
				g_Neuz.m_camera.m_fRoty += 2;
			else
				g_Neuz.m_camera.m_fRoty -= 2;

			if (g_Option.Camera_AngleScroll)
			{
				if (g_Neuz.m_camera.m_fRoty > 80 - g_Neuz.m_camera.m_fZoom * 4)
					g_Neuz.m_camera.m_fRoty = 80 - g_Neuz.m_camera.m_fZoom * 4;
			}
#else
			g_Neuz.m_camera.m_fRoty -= 2;
			if (g_Neuz.m_camera.m_fRoty > 80 - g_Neuz.m_camera.m_fZoom * 4)
				g_Neuz.m_camera.m_fRoty = 80 - g_Neuz.m_camera.m_fZoom * 4;
#endif
```
- Camera.cpp
```cpp
#ifdef __CameraOptions 
	//m_vLookAt.y += 0.34f; // Lower's the camera position
	m_vLookAt.y += 0.4f;
	m_fCurRoty = m_fRoty;
	if (g_Option.Camera_AngleScroll)
		m_fCurRoty += m_fZoom * 4.0f;
#else
	m_vLookAt.y += 0.4f;
	m_fCurRoty = m_fRoty + m_fZoom * 4;

#endif
```

> Note: m_bRollEffect might need to be merged with Camera_AngleScroll.



## License
---
_FlyFF is an mmorpg created by GalaNet/Aeonsoft. This implementation was done by myself:_
| Account  | Site |
| -------- | ------- |
| Kiaemi. | Discord |
| Velvet | RageZone |
| M¿dScientist | Elitepvpers |

