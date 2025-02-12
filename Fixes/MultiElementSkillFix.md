## Making Multi-Elemental Skills Get Bonuses From Mastery%
_Elementors have multi-elemented skills. These takes the multi-elemented ST defines, and adds them in to the list to get the mastery%. This updates it so it applies the mastery of each element within the multi-element ST_

- **MoverAttack.cpp**
  ```CPP
	  case ST_EARTH:	
		  fRatio	= GetParam( DST_MASTRY_EARTH, 0 ) / 100.0f;
		  nATK	= (int)( nATK + (nATK * fRatio) );
		  break;
    // here
		case ST_FIREEARTH: fRatio = (GetParam(DST_MASTRY_FIRE, 0) / 100.0f) + (GetParam(DST_MASTRY_EARTH, 0) / 100.0f); nATK = (int)(nATK + (nATK * fRatio)); break;
		case ST_EARTHWATER: fRatio = (GetParam(DST_MASTRY_WATER, 0) / 100.0f) + (GetParam(DST_MASTRY_EARTH, 0) / 100.0f); nATK = (int)(nATK + (nATK * fRatio)); break;
		case ST_ELECWIND: fRatio = (GetParam(DST_MASTRY_WIND, 0) / 100.0f) + (GetParam(DST_MASTRY_ELECTRICITY, 0) / 100.0f); nATK = (int)(nATK + (nATK * fRatio)); break;
		case ST_EARTHWIND: fRatio = (GetParam(DST_MASTRY_WIND, 0) / 100.0f) + (GetParam(DST_MASTRY_EARTH, 0) / 100.0f); nATK = (int)(nATK + (nATK * fRatio)); break;
  ```
