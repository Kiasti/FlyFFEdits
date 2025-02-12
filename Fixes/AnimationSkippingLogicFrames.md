
## Animation skipping logic frames for dps.
---
When the players speed is too quick, frames for logic is skipped. Ideally, you'd want to return all instances between fOldFrm and fNumFrm instead of just one to handle all skipped instances of the MA_HIT flag and handle the damage/combat.
- bone.h
	```cpp
	DWORD IsAttrHit(const float fOldFrm, const float fNumFrm) const
	{ 
	#ifdef __fixes
		unsigned int old = static_cast<unsigned int>(fOldFrm) + 1;
		while (old < static_cast<unsigned int>(fNumFrm))
		{
			MOTION_ATTR *pAttr = &m_pAttr[old];
			if (pAttr->m_dwAttr & MA_HIT)
			{
				if (fOldFrm < pAttr->m_fFrame && pAttr->m_fFrame <= fNumFrm)
					return pAttr->m_dwAttr;
			}
			++old;
		}
	#endif
		MOTION_ATTR	 *pAttr = &m_pAttr[(int)fNumFrm];
		if (pAttr->m_dwAttr & MA_HIT)
		{
			if (fOldFrm < pAttr->m_fFrame && pAttr->m_fFrame <= fNumFrm)	// ÀÌÀü ÇÁ·¹ÀÓÀÌ¶û ÇöÀç ÇÁ·¹ÀÓ »çÀÌ¿¡ Å¸Á¡ÀÌ µé¾îÀÖ¾ú´Â°¡.
				return pAttr->m_dwAttr;
		}
		return 0;
	}
	```

<br>

## Thank you
---
_FlyFF is an mmorpg created by GalaNet/Aeonsoft. This implementation was done by myself:_
| Account  | Site |
| -------- | ------- |
| Kiaemi. | Discord |
| Velvet | RageZone |
| M¿dScientist | Elitepvpers |



