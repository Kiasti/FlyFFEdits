## Adding an options for Farthest Plane Idea
_This can drastically increase or decrease player land view range._

- **HwOption.cpp**
  _Load_
	You're able to load a float with the CScanner class to load anything for the farplane range.
  ```CPP
		else if (scan.Token == _T("farPlane"))
			CWorld::m_fFarPlane = scan.GetFloat();
	 ```
	or you're able to do something like the following where the user only has a few options to chose from.
	```cpp
	else if (scan.Token == _T("FarPlane"))
	{
		FarPlane = scan.GetNumber(); // Add the variable to .h, etc.
		switch (FarPlane)
		{
			case 0: CWorld::m_fFarPlane = 4096.f; break;
			case 1: CWorld::m_fFarPlane = 2048.f; break;
			default:
			case 2: CWorld::m_fFarPlane = 1024.f; break;
			case 3: CWorld::m_fFarPlane = 512.f; break;
			case 4: CWorld::m_fFarPlane = 256.f; break;
		}
		if (FarPlane > 4)
			FarPlane = 2;
	}
	```

  _Save_
  ```CPP
    _ftprintf(fp, _T("farPlane %f\n"), CWorld::m_fFarPlane);
  ```
  
  >Note: If you are using the latter, you will require storing the "FarPlane" option separately from the actual farplane value, this way you can save the farplane option instead of CWorld::m_fFarPlane. (Or you can use find out the option upon saving instead)

