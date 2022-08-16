# Box Preview
_Box Preview/Opener is a system to slow down box opening slightly but also give information to what the box contains. Whether it be a gift box, or a pack box, both information is shown. This also includes the percentages to acquire an item. If an item has a + modifer, that is also shown in the tooltip._

---
## Features
- Preview boxes and rates.


## Adding parts to the source
---

- **User.cpp** _(Worldserver)_
    find and replace __this function__
    ```CPP
    #ifdef __BoxPreview
    void CUser::DoUsePackItem(CItemElem* pItemElem, const _PACKITEMELEM* pPackItemElem)
    {
    	time_t t = 0;
    	if( pPackItemElem->span)	// minutes
    	{
    		CTime time	= CTime::GetCurrentTime() + CTimeSpan( 0, 0, pPackItemElem->span, 0 );
    		t	= static_cast<time_t>(time.GetTime());
    	}
    
    	if( m_Inventory.GetEmptyCount() >= pPackItemElem->size() )
    	{
    		for (const auto& iter : pPackItemElem->itemList)
    		{
    			CItemElem itemElem;
    			itemElem.m_dwItemId	= iter.pb.adwItem;
    			itemElem.SetAbilityOption(iter.pb.anAbilityOption);
    			itemElem.m_nItemNum	= iter.pb.anNum;
    			itemElem.m_bCharged		= itemElem.GetProp()->bCharged;
    			itemElem.m_dwKeepTime	= static_cast<DWORD>(t);
    
    			if( pItemElem->IsBinds() )
    				itemElem.SetFlag( CItemElem::binds );
    
    			if( CreateItem( &itemElem ) )
    			{
    				AddDefinedText( TID_GAME_REAPITEM, "\"%s\"", itemElem.GetProp()->szName );
    				g_DPSrvr.PutCreateItemLog( this, &itemElem , "E", "PACK" );
    			}
    			else
    			{
    			}
    		}
    		OnAfterUseItem( pItemElem->GetProp() );
    		UpdateItem( (BYTE)( pItemElem->m_dwObjId ), UI_NUM, pItemElem->m_nItemNum - 1 );
    	}
    	else
    	{
    		AddDefinedText( TID_GAME_LACKSPACE );			
    	}
    }
    #else
    void CUser::DoUsePackItem(CItemElem* pItemElem, PPACKITEMELEM pPackItemElem)
    {
    	time_t t	= 0;
    	if( pPackItemElem->nSpan )	// minutes
    	{
    		CTime time	= CTime::GetCurrentTime() + CTimeSpan( 0, 0, pPackItemElem->nSpan, 0 );
    		t	= (time_t)( time.GetTime() );
    	}
    
    	if( m_Inventory.GetEmptyCount() >= pPackItemElem->nSize )
    	{
    		for( int i = 0; i < pPackItemElem->nSize; i++ )
    		{
    			CItemElem itemElem;
    			itemElem.m_dwItemId	= pPackItemElem->adwItem[i];
    			itemElem.SetAbilityOption( pPackItemElem->anAbilityOption[i] );
    			itemElem.m_nItemNum	= pPackItemElem->anNum[i];
    			itemElem.m_bCharged		= itemElem.GetProp()->bCharged;
    			itemElem.m_dwKeepTime	= (DWORD)t;
    
    //			if( pItemElem->IsFlag( CItemElem::binds ) )
    			if( pItemElem->IsBinds() )
    				itemElem.SetFlag( CItemElem::binds );
    
    			if( CreateItem( &itemElem ) )
    			{
    				AddDefinedText( TID_GAME_REAPITEM, "\"%s\"", itemElem.GetProp()->szName );
    				g_DPSrvr.PutCreateItemLog( this, &itemElem , "E", "PACK" );
    //				ItemProp* pItemProp		= itemElem.GetProp();
    //				if( pItemProp->dwSfxObj3 != -1 )
    //					g_UserMng.AddCreateSfxObj( this, pItemProp->dwSfxObj3, GetPos().x, GetPos().y, GetPos().z );
    			}
    			else
    			{
    				// critical err
    			}
    		}
    		OnAfterUseItem( pItemElem->GetProp() );
    		UpdateItem( (BYTE)( pItemElem->m_dwObjId ), UI_NUM, pItemElem->m_nItemNum - 1 );
    	}
    	else
    	{
    		AddDefinedText( TID_GAME_LACKSPACE );			
    	}
    }
    #endif
    ```
    find and replace in __CUser::OnDoUseItem__
    ```CPP
    #ifdef __BoxPreview
    		const _PACKITEMELEM* const packItem = CPackItem::GetInstance()->getPackItem(dwItemId);
    		if (packItem)
    		{
    			DoUsePackItem(pItemElem, packItem);
    		}
    #else
    		PPACKITEMELEM pPackItemElem	= CPackItem::GetInstance()->Open( dwItemId );
    		if( pPackItemElem )
    		{
    			DoUsePackItem( pItemElem, pPackItemElem );
    			return;
    		}
    #endif
    #ifdef __BoxPreview
    		if (DoUseGiftbox(pItemElem, dwItemId))
    			return;
    #endif
    	}
    	else
    	{
    		pItemElem = NULL;
    		dwItemId = 0;
    	}
    
    	DoUseItem( dwData, objid, nPart );
    
    #ifndef __BoxPreview
    	if( DoUseGiftbox( pItemElem, dwItemId ) )
    		return;
    #endif
    ```  
- **User.h** _(Worldserver)_
    find and replace
    ```CPP
    #ifdef __BoxPreview
    	void DoUsePackItem(CItemElem* pItemElem, const _PACKITEMELEM* pPackItemElem);
    #else
    	void			DoUsePackItem( CItemElem* pItemElem, PPACKITEMELEM pPackItemElem );
    #endif
    ```
- **Project.cpp** _(Common)_
    ```CPP
    #ifdef __BoxPreview
    	LoadGiftbox("propGiftbox.inc");
    	LoadPackItem("propPackItem.inc");
    #endif
    #ifdef __WORLDSERVER
    	LoadConstant( "Constant.inc" );
    	LoadPropGuildQuest( "propGuildQuest.inc" );
    	LoadPropPartyQuest( "propPartyQuest.inc" );
    
    	LoadDropEvent( "propDropEvent.inc" );
    #ifndef __BoxPreview
    	LoadGiftbox( "propGiftbox.inc" );
    	LoadPackItem( "propPackItem.inc" );
    #endif
    ```
    Find the start of what is in the #else and replace the start to end with this:
    ```CPP
    #ifdef __BoxPreview
    CGiftboxMan::CGiftboxMan() = default;
    CGiftboxMan::~CGiftboxMan() = default;
    
    CGiftboxMan* CGiftboxMan::GetInstance()
    {
    	static CGiftboxMan sGiftboxMan;
    	return &sGiftboxMan;
    }
    
    bool CGiftboxMan::isGiftbox(const unsigned long itemId)
    {
    	if (itemList.find(itemId) != itemList.end())
    		return true;
    	return false;
    }
    
    std::vector<boxItems>* CGiftboxMan::getInternalItemList(const unsigned long mainId)
    {
    	_GIFTBOX* target = getGiftbox(mainId);
    	if (target)
    		return &target->itemList;
    	return nullptr;
    }
    
    _GIFTBOX* CGiftboxMan::getGiftbox(const unsigned long itemId)
    {
    	std::map<unsigned long, _GIFTBOX>::iterator iter = itemList.find(itemId);
    	if (iter != itemList.end())
    		return &iter->second;
    	return nullptr;
    }
    
    std::pair<std::map<unsigned long, _GIFTBOX>::iterator, bool> CGiftboxMan::insert(unsigned long id, _GIFTBOX gb)
    {
    	return itemList.insert(std::make_pair(id, gb));
    }
    
    #ifdef __WORLDSERVER
    bool CGiftboxMan::open(const unsigned long itemId, PGIFTBOXRESULT pGiftboxResult)
    {
    	const _GIFTBOX* const giftBox = getGiftbox(itemId);
    	if (!giftBox)
    		return false;
    
    	const unsigned long calculateRandom = xRandom(giftBox->nSum); 
    	unsigned long lowPercent = 0;
    	for (size_t i = 0; i < giftBox->size(); ++i)
    	{
    		lowPercent += giftBox->itemList[i].gb.adwProbability;
    		if (calculateRandom < lowPercent)
    		{
    			pGiftboxResult->dwItem = giftBox->itemList[i].gb.adwItem;
    			pGiftboxResult->nNum = giftBox->itemList[i].gb.anNum;
    			pGiftboxResult->nFlag = giftBox->itemList[i].gb.anFlag;
    			pGiftboxResult->nSpan = giftBox->itemList[i].gb.anSpan;
    			pGiftboxResult->nAbilityOption = giftBox->itemList[i].gb.anAbilityOption;
    			return true;
    		}
    	}
    	return false;
    }
    #endif
    
    
    BOOL CProject::LoadGiftbox(const char* lpszFileName) const
    {
    	CScript s;
    	if (!s.Load(lpszFileName))
    		return false;
    
    	std::vector<boxItems> itemListPerGiftBox;
    	CGiftboxMan* singletonInstance = CGiftboxMan::GetInstance();
    
    	boxItems iGiftBox{};
    	unsigned long loadingId;
    	s.GetToken();
    	while (s.tok != FINISHED)
    	{
    		if (s.Token == _T("GiftBox"))
    		{
    			loadingId = s.GetNumber();
    
    			_GIFTBOX giftBox;
    			giftBox.dwGiftbox = loadingId;
    			s.GetToken();
    			iGiftBox.gb.adwItem = s.GetNumber();
    			while (*s.token != '}')
    			{
    				iGiftBox.gb.adwProbability = s.GetNumber();
    				giftBox.nSum += iGiftBox.gb.adwProbability;
    				iGiftBox.gb.anNum = s.GetNumber();
    
    				giftBox.itemList.emplace_back(iGiftBox);
    				iGiftBox.gb.adwItem = s.GetNumber();
    			}
    			singletonInstance->insert(loadingId, std::move(giftBox));
    		}
    		else if (s.Token == _T("GiftBox2"))
    		{
    			loadingId = s.GetNumber();
    
    			_GIFTBOX giftBox;
    			giftBox.dwGiftbox = loadingId;
    
    			s.GetToken();
    			iGiftBox.gb.adwItem = s.GetNumber();
    			while (*s.token != '}')
    			{
    				iGiftBox.gb.adwProbability = s.GetNumber();
    				giftBox.nSum += iGiftBox.gb.adwProbability;
    				iGiftBox.gb.anNum = s.GetNumber();
    				giftBox.itemList.emplace_back(iGiftBox);
    				iGiftBox.gb.adwItem = s.GetNumber();
    			}
    			singletonInstance->insert(loadingId, std::move(giftBox));
    		}
    		else if (s.Token == _T("GiftBox3"))
    		{
    			loadingId = s.GetNumber();
    
    			_GIFTBOX giftBox;
    			giftBox.dwGiftbox = loadingId;
    
    			s.GetToken();
    			iGiftBox.gb.adwItem = s.GetNumber();
    			while (*s.token != '}')
    			{
    				iGiftBox.gb.adwProbability = s.GetNumber();
    				giftBox.nSum += iGiftBox.gb.adwProbability;
    				iGiftBox.gb.anNum = s.GetNumber();
    				iGiftBox.gb.anFlag = s.GetNumber();
    				giftBox.itemList.emplace_back(iGiftBox);
    				iGiftBox.gb.adwItem = s.GetNumber();
    			}
    			singletonInstance->insert(loadingId, std::move(giftBox));
    		}
    		else if (s.Token == _T("GiftBox4") || s.Token == _T("GiftBox5"))
    		{
    			loadingId = s.GetNumber();
    
    			_GIFTBOX giftBox;
    			giftBox.dwGiftbox = loadingId;
    
    			s.GetToken();
    			iGiftBox.gb.adwItem = s.GetNumber();
    			while (*s.token != '}')
    			{
    				iGiftBox.gb.adwProbability = s.GetNumber();
    				giftBox.nSum += iGiftBox.gb.adwProbability;
    				iGiftBox.gb.anNum = s.GetNumber();
    				iGiftBox.gb.anFlag = s.GetNumber();
    				iGiftBox.gb.anSpan = s.GetNumber();
    
    				giftBox.itemList.emplace_back(iGiftBox);
    				iGiftBox.gb.adwItem = s.GetNumber();
    			}
    			singletonInstance->insert(loadingId, std::move(giftBox));
    		}
    		else if (s.Token == _T("GiftBox6"))
    		{
    			loadingId = s.GetNumber();
    
    			_GIFTBOX giftBox;
    			giftBox.dwGiftbox = loadingId;
    
    			s.GetToken();
    			iGiftBox.gb.adwItem = s.GetNumber();
    			while (*s.token != '}')
    			{
    				iGiftBox.gb.adwProbability = s.GetNumber();
    				giftBox.nSum += iGiftBox.gb.adwProbability;
    				iGiftBox.gb.anNum = s.GetNumber();
    				iGiftBox.gb.anFlag = s.GetNumber();
    				iGiftBox.gb.anSpan = s.GetNumber();
    				iGiftBox.gb.anAbilityOption = s.GetNumber();
    
    				giftBox.itemList.emplace_back(iGiftBox);
    				iGiftBox.gb.adwItem = s.GetNumber();
    			}
    			singletonInstance->insert(loadingId, std::move(giftBox));
    		}
    		s.GetToken();
    	}
    	return TRUE;
    }
    
    CPackItem::CPackItem() = default;
    CPackItem* CPackItem::GetInstance(void)
    {
    	static CPackItem sPackItem;
    	return &sPackItem;
    }
    
    bool CPackItem::isPackItem(const unsigned long itemId)
    {
    	return itemList.find(itemId) != itemList.end();
    }
    _PACKITEMELEM* CPackItem::getPackItem(const unsigned long itemId)
    {
    	std::map<unsigned long, _PACKITEMELEM>::iterator iter = itemList.find(itemId);
    	if (iter != itemList.end())
    		return &iter->second;
    	return nullptr;
    }
    
    BOOL CProject::LoadPackItem(const char* lpszFileName) const
    {
    	CScript s;
    	if (s.Load(lpszFileName) == FALSE)
    		return FALSE;
    
    	boxItems internalBox{};
    	CPackItem* packItemSingleton = CPackItem::GetInstance();
    
    	s.GetToken();
    	while (s.tok != FINISHED)
    	{
    		if (s.Token == _T("PackItem"))
    		{
    			unsigned long loadingId = s.GetNumber();
    			_PACKITEMELEM packElem;
    			packElem.dwPackItem = loadingId;
    			packElem.span = s.GetNumber();
    			s.GetToken();	// {
    			internalBox.pb.adwItem = s.GetNumber();
    			while (*s.token != '}')
    			{
    				internalBox.pb.anAbilityOption = s.GetNumber();
    				internalBox.pb.anNum = s.GetNumber();
    				packElem.itemList.emplace_back(internalBox);
    				internalBox.pb.adwItem = s.GetNumber();
    			}
    			packItemSingleton->getItemList().insert(std::make_pair(loadingId, std::move(packElem)));
    		}
    		s.GetToken();
    	}
    	return TRUE;
    }
    
    // old
    #else
    #ifdef __WORLDSERVER
    CGiftboxMan::CGiftboxMan()
    {
    #ifndef __STL_GIFTBOX_VECTOR
    	m_nSize = 0;
    	memset(&m_giftbox, 0, sizeof(m_giftbox));
    #endif // __STL_GIFTBOX_VECTOR
    	/*
    #ifdef __GIFTBOX0213
    	m_nQuery	= 0;
    #endif	// __GIFTBOX0213
    	*/
    }
    
    CGiftboxMan* CGiftboxMan::GetInstance(void)
    {
    	static CGiftboxMan sGiftboxMan;
    	return &sGiftboxMan;
    }
    
    /*
    #ifdef __GIFTBOX0213
    BOOL CGiftboxMan::AddItem( DWORD dwGiftbox, DWORD dwItem, DWORD dwProbability, int nNum, BYTE nFlag, int nTotal )
    #else	// __GIFTBOX0213
    */
    BOOL CGiftboxMan::AddItem(DWORD dwGiftbox, DWORD dwItem, DWORD dwProbability, int nNum, BYTE nFlag, int nSpan, int     nAbilityOption)
    //#endif	// __GIFTBOX0213
    {
    	map<DWORD, int>::iterator i = m_mapIdx.find(dwGiftbox);
    	int nIdx1 = 0;
    	if (i != m_mapIdx.end())
    	{
    		nIdx1 = i->second;
    	}
    	else
    	{
    #ifdef __STL_GIFTBOX_VECTOR
    		nIdx1 = m_vGiftBox.size();
    		GIFTBOX giftBox;
    		memset(&giftBox, 0, sizeof(GIFTBOX));
    		m_vGiftBox.push_back(giftBox);
    #else // __STL_GIFTBOX_VECTOR
    		nIdx1 = m_nSize++;
    #endif // __STL_GIFTBOX_VECTOR
    		m_mapIdx.insert(map<DWORD, int>::value_type(dwGiftbox, nIdx1));
    	}
    
    #ifdef __STL_GIFTBOX_VECTOR
    	m_vGiftBox[nIdx1].dwGiftbox = dwGiftbox;
    	int nIdx2 = m_vGiftBox[nIdx1].nSize++;
    	m_vGiftBox[nIdx1].adwItem[nIdx2] = dwItem;
    	m_vGiftBox[nIdx1].anNum[nIdx2] = nNum;
    	m_vGiftBox[nIdx1].anFlag[nIdx2] = nFlag;
    	m_vGiftBox[nIdx1].anSpan[nIdx2] = nSpan;
    	m_vGiftBox[nIdx1].anAbilityOption[nIdx2] = nAbilityOption;
    
    	m_vGiftBox[nIdx1].nSum += dwProbability;
    	m_vGiftBox[nIdx1].adwProbability[nIdx2] = m_vGiftBox[nIdx1].nSum;
    #else // __STL_GIFTBOX_VECTOR
    	if (m_nSize >= MAX_GIFTBOX)
    	{
    		OutputDebugString("TOO MANY GIFTBOX\n");
    		return FALSE;
    	}
    
    	m_giftbox[nIdx1].dwGiftbox = dwGiftbox;
    	int nIdx2 = m_giftbox[nIdx1].nSize++;
    	m_giftbox[nIdx1].adwItem[nIdx2] = dwItem;
    	m_giftbox[nIdx1].anNum[nIdx2] = nNum;
    	m_giftbox[nIdx1].anFlag[nIdx2] = nFlag;
    	m_giftbox[nIdx1].anSpan[nIdx2] = nSpan;
    	m_giftbox[nIdx1].anAbilityOption[nIdx2] = nAbilityOption;
    	/*
    #ifdef __GIFTBOX0213
    	m_giftbox[nIdx1].anTotal[nIdx2]	= nTotal;
    	if( nTotal > 0 )
    		m_giftbox[nIdx1].bGlobal	= TRUE;
    #endif	// __GIFTBOX0213
    		*/
    	m_giftbox[nIdx1].nSum += dwProbability;
    	m_giftbox[nIdx1].adwProbability[nIdx2] = m_giftbox[nIdx1].nSum;
    #endif // __STL_GIFTBOX_VECTOR
    	return TRUE;
    }
    
    /*
    #ifdef __GIFTBOX0213
    BOOL CGiftboxMan::Open( DWORD dwGiftbox, LPDWORD pdwItem, int* pnNum, u_long idPlayer, CItemElem* pItemElem )
    #else	// __GIFTBOX0213
    */
    BOOL CGiftboxMan::Open(DWORD dwGiftbox, PGIFTBOXRESULT pGiftboxResult)
    //#endif	// __GIFTBOX0213
    {
    	DWORD dwRand = xRandom(1000000);
    	map<DWORD, int>::iterator i = m_mapIdx.find(dwGiftbox);
    	if (i == m_mapIdx.end())
    		return 0;
    	int nIdx = i->second;
    #ifdef __STL_GIFTBOX_VECTOR
    	PGIFTBOX pBox = &m_vGiftBox[nIdx];
    #else // __STL_GIFTBOX_VECTOR
    	PGIFTBOX pBox = &m_giftbox[nIdx];
    #endif // __STL_GIFTBOX_VECTOR
    
    	int low = 0;
    	for (int j = 0; j < pBox->nSize; j++)
    	{
    		if (dwRand >= (DWORD)(low) && dwRand < pBox->adwProbability[j])
    		{
    			/*
    #ifdef __GIFTBOX0213
    			if( pBox->anTotal[j] > 0 )
    			{
    				Query( CDPAccountClient::GetInstance(), dwGiftbox, pBox->adwItem[j], pBox->anNum[j], idPlayer,     pItemElem );
    				return FALSE;
    			}
    #endif	// __GIFTBOX0213
    			*/
    //			*pdwItem	= pBox->adwItem[j];
    //			*pnNum	= pBox->anNum[j];
    //			*pnFlag	= pBox->anFlag[j];
    //			*pnSpan	= pBox->anSpan[j];
    			pGiftboxResult->dwItem = pBox->adwItem[j];
    			pGiftboxResult->nNum = pBox->anNum[j];
    			pGiftboxResult->nFlag = pBox->anFlag[j];
    			pGiftboxResult->nSpan = pBox->anSpan[j];
    			pGiftboxResult->nAbilityOption = pBox->anAbilityOption[j];
    			return TRUE;
    		}
    	}
    	return FALSE;
    }
    
    /*
    #ifdef __GIFTBOX0213
    BOOL CGiftboxMan::OpenLowest( DWORD dwGiftbox, LPDWORD pdwItem, int* pnNum )
    {
    	map<DWORD, int>::iterator i		= m_mapIdx.find( dwGiftbox );
    	if( i == m_mapIdx.end() )
    		return FALSE;
    	int nIdx	= i->second;
    	PGIFTBOX pBox	= &m_giftbox[nIdx];
    
    	int nLowest	= pBox->nSize - 1;
    	if( nLowest < 0 )
    		return FALSE;
    	*pdwItem	= pBox->adwItem[nLowest];
    	*pnNum	= pBox->anNum[nLowest];
    	return TRUE;
    }
    #endif	// __GIFTBOX0213
    */
    
    void CGiftboxMan::Verify(void)
    {
    #ifdef __STL_GIFTBOX_VECTOR
    	for (DWORD i = 0; i < m_vGiftBox.size(); i++)
    	{
    		TRACE("GIFTBOX : %d, %d\n", m_vGiftBox[i].dwGiftbox, m_vGiftBox[i].nSum);
    		m_vGiftBox[i].adwProbability[m_vGiftBox[i].nSize - 1] += (1000000 - m_vGiftBox[i].nSum);
    	}
    #else // __STL_GIFTBOX_VECTOR
    	for (int i = 0; i < m_nSize; i++)
    	{
    		TRACE("GIFTBOX : %d, %d\n", m_giftbox[i].dwGiftbox, m_giftbox[i].nSum);
    		m_giftbox[i].adwProbability[m_giftbox[i].nSize - 1] += (1000000 - m_giftbox[i].nSum);
    	}
    #endif // __STL_GIFTBOX_VECTOR
    }
    
    BOOL CProject::LoadGiftbox(LPCTSTR lpszFileName)
    {
    	CScript s;
    	if (s.Load(lpszFileName) == FALSE)
    		return FALSE;
    
    	DWORD dwGiftbox, dwItem, dwProbability;
    	int	nNum;
    	BYTE	nFlag;
    	int	nSpan;
    	int	nAbilityOption;
    
    	s.GetToken();	// subject or FINISHED
    	while (s.tok != FINISHED)
    	{
    		if (s.Token == _T("GiftBox"))
    		{
    			dwGiftbox = s.GetNumber();
    			s.GetToken();	// {
    			dwItem = s.GetNumber();
    			while (*s.token != '}')
    			{
    				dwProbability = s.GetNumber();
    				nNum = s.GetNumber();
    				if (!CGiftboxMan::GetInstance()->AddItem(dwGiftbox, dwItem, dwProbability * 100, nNum))
    					return FALSE;
    				dwItem = s.GetNumber();
    			}
    		}
    		else if (s.Token == _T("GiftBox2"))
    		{
    			dwGiftbox = s.GetNumber();
    			s.GetToken();	// {
    			dwItem = s.GetNumber();
    			while (*s.token != '}')
    			{
    				dwProbability = s.GetNumber();
    				nNum = s.GetNumber();
    				if (!CGiftboxMan::GetInstance()->AddItem(dwGiftbox, dwItem, dwProbability, nNum))
    					return FALSE;
    				dwItem = s.GetNumber();
    			}
    		}
    		else if (s.Token == _T("GiftBox3"))
    		{
    			dwGiftbox = s.GetNumber();
    			s.GetToken();	// {
    			dwItem = s.GetNumber();
    			while (*s.token != '}')
    			{
    				dwProbability = s.GetNumber();
    				nNum = s.GetNumber();
    				nFlag = s.GetNumber();
    				if (!CGiftboxMan::GetInstance()->AddItem(dwGiftbox, dwItem, dwProbability * 100, nNum, nFlag))
    					return FALSE;
    				dwItem = s.GetNumber();
    			}
    		}
    		else if (s.Token == _T("GiftBox4") || s.Token == _T("GiftBox5"))
    		{
    			DWORD dwPrecision = (s.Token == _T("GiftBox4") ? 100 : 10);
    			dwGiftbox = s.GetNumber();
    			s.GetToken();	// {
    			dwItem = s.GetNumber();
    			while (*s.token != '}')
    			{
    				dwProbability = s.GetNumber();
    				nNum = s.GetNumber();
    				nFlag = s.GetNumber();
    				nSpan = s.GetNumber();	// ±â°£
    
    				if (!CGiftboxMan::GetInstance()->AddItem(dwGiftbox, dwItem, dwProbability * dwPrecision, nNum,     nFlag, nSpan))
    					return FALSE;
    				dwItem = s.GetNumber();
    			}
    		}
    		else if (s.Token == _T("GiftBox6"))
    		{
    			dwGiftbox = s.GetNumber();
    			s.GetToken();	// {
    			dwItem = s.GetNumber();
    			while (*s.token != '}')
    			{
    				dwProbability = s.GetNumber();
    				nNum = s.GetNumber();
    				nFlag = s.GetNumber();
    				nSpan = s.GetNumber();	// ±â°£
    				nAbilityOption = s.GetNumber();	// +
    
    				if (!CGiftboxMan::GetInstance()->AddItem(dwGiftbox, dwItem, dwProbability * 10, nNum, nFlag,     nSpan, nAbilityOption))
    					return FALSE;
    				dwItem = s.GetNumber();
    			}
    		}
    		/*
    #ifdef __GIFTBOX0213
    		else if( s.Token == _T( "GlobalGiftbox" ) )
    		{
    			int nTotal;
    			dwGiftbox	= s.GetNumber();
    			s.GetToken();	// {
    			dwItem	= s.GetNumber();
    			while( *s.token != '}' )
    			{
    				dwProbability	= s.GetNumber();
    				nNum	= s.GetNumber();
    				nTotal	= s.GetNumber();
    
    				if( !CGiftboxMan::GetInstance()->AddItem( dwGiftbox, dwItem, dwProbability, nNum, 0, nTotal ) )
    					return FALSE;
    				dwItem	= s.GetNumber();
    			}
    		}
    #endif	// __GIFTBOX0213
    		*/
    		s.GetToken();
    	}
    	CGiftboxMan::GetInstance()->Verify();
    	return TRUE;
    }
    
    /*
    #ifdef __GIFTBOX0213
    void CGiftboxMan::Upload( CDPMng* pdp )
    {
    	CAr ar;
    	ar << PACKETTYPE_GLOBALGIFTBOX;
    	u_long uOffset	= ar.GetOffset();
    	int nSize	= 0;
    	ar << nSize;
    	for( int i = 0; i < m_nSize; i++ )
    	{
    		if( m_giftbox[i].bGlobal )
    		{
    			ar << m_giftbox[i].dwGiftbox;
    			ar << m_giftbox[i].nSize;
    			for( int j = 0; j < m_giftbox[i].nSize; j++ )
    			{
    				ar << m_giftbox[i].adwItem[j];
    				ar << m_giftbox[i].anTotal[j];
    			}
    			nSize++;
    		}
    	}
    	int nBufSize;
    	LPBYTE lpBuf	= ar.GetBuffer( &nBufSize );
    	*(UNALIGNED int*)( lpBuf + uOffset )	= nSize;
    	pdp->Send( lpBuf, nBufSize, DPID_SERVERPLAYER );
    }
    
    void CGiftboxMan::Query( CDPMng* pdp, DWORD dwGiftbox, DWORD dwItem, int nNum, u_long idPlayer, CItemElem*     pItemElem )
    {
    	if( pItemElem->m_nQueryGiftbox > 0 )
    		return;
    
    	CAr ar;
    	ar << PACKETTYPE_QUERYGLOBALGIFTBOX;
    	pItemElem->m_nQueryGiftbox	= ++m_nQuery;
    	ar << idPlayer << dwGiftbox << dwItem << nNum << pItemElem->m_dwObjId << pItemElem->m_nQueryGiftbox;
    	int nBufSize;
    	LPBYTE lpBuf	= ar.GetBuffer( &nBufSize );
    	pdp->Send( lpBuf, nBufSize, DPID_SERVERPLAYER );
    }
    
    void CGiftboxMan::Restore( CDPMng* pdp, DWORD dwGiftbox, DWORD dwItem )
    {
    	CAr ar;
    	ar << PACKETTYPE_RESTOREGLOBALGIFTBOX;
    	ar << dwGiftbox << dwItem;
    	int nBufSize;
    	LPBYTE lpBuf	= ar.GetBuffer( &nBufSize );
    	pdp->Send( lpBuf, nBufSize, DPID_SERVERPLAYER );
    }
    #endif	// __GIFTBOX0213
    */
    #endif	// __WORLDSERVER
    
    CPackItem::CPackItem()
    {
    	m_nSize = 0;
    	memset(&m_packitem, 0, sizeof(m_packitem));
    }
    
    CPackItem* CPackItem::GetInstance(void)
    {
    	static CPackItem sPackItem;
    	return &sPackItem;
    }
    
    BOOL CPackItem::AddItem(DWORD dwPackItem, DWORD dwItem, int nAbilityOption, int nNum)
    {
    	map<DWORD, int>::iterator i = m_mapIdx.find(dwPackItem);
    	int nIdx1 = 0;
    	if (i != m_mapIdx.end())
    	{
    		nIdx1 = i->second;
    	}
    	else
    	{
    		nIdx1 = m_nSize++;
    		m_mapIdx.insert(map<DWORD, int>::value_type(dwPackItem, nIdx1));
    	}
    
    	if (m_nSize >= MAX_PACKITEM)
    	{
    		OutputDebugString("TOO MANY PACKITEM\n");
    		return FALSE;
    	}
    
    #ifdef __GAMEGUARD
    	if (m_packitem[nIdx1].nSize >= MAX_ITEM_PER_PACK)
    #else
    	if (m_packitem[nIdx1].nSize == MAX_ITEM_PER_PACK)
    #endif
    	{
    		OutputDebugString("TOO MANY ITEM PER PACK\n");
    		return FALSE;
    	}
    
    	m_packitem[nIdx1].dwPackItem = dwPackItem;
    	int nIdx2 = m_packitem[nIdx1].nSize++;
    	m_packitem[nIdx1].adwItem[nIdx2] = dwItem;
    	m_packitem[nIdx1].anAbilityOption[nIdx2] = nAbilityOption;
    	m_packitem[nIdx1].anNum[nIdx2] = nNum;
    	return TRUE;
    }
    
    PPACKITEMELEM CPackItem::Open(DWORD dwPackItem)
    {
    	map<DWORD, int>::iterator i = m_mapIdx.find(dwPackItem);
    	if (i != m_mapIdx.end())
    		return &m_packitem[i->second];
    	return NULL;
    }
    
    BOOL CProject::LoadPackItem(LPCTSTR lpszFileName)
    {
    	CScript s;
    	if (s.Load(lpszFileName) == FALSE)
    		return FALSE;
    
    	DWORD dwPackItem, dwItem;
    	int nAbilityOption, nNum;
    	int	nSpan;
    
    	char lpOutputString[100] = { 0, };
    	OutputDebugString("packItem\n");
    	OutputDebugString("----------------------------------------\n");
    	s.GetToken();	// subject or FINISHED
    	while (s.tok != FINISHED)
    	{
    		if (s.Token == _T("PackItem"))
    		{
    			dwPackItem = s.GetNumber();
    			nSpan = s.GetNumber();
    			s.GetToken();	// {
    			dwItem = s.GetNumber();
    			while (*s.token != '}')
    			{
    				nAbilityOption = s.GetNumber();
    				nNum = s.GetNumber();
    
    				sprintf(lpOutputString, "%d\n", dwPackItem);
    				OutputDebugString(lpOutputString);
    				if (!CPackItem::GetInstance()->AddItem(dwPackItem, dwItem, nAbilityOption, nNum))
    					return FALSE;
    				dwItem = s.GetNumber();
    			}
    			PPACKITEMELEM pPackItemElem = CPackItem::GetInstance()->Open(dwPackItem);
    			if (pPackItemElem)
    				pPackItemElem->nSpan = nSpan;
    		}
    		s.GetToken();
    	}
    	OutputDebugString("----------------------------------------\n");
    	return TRUE;
    }
    #endif
    ```
- **Project.h** _(common)_
    find and replace
    ```CPP
    #ifdef __BoxPreview
    struct internalGiftBoxItem
    {
    	unsigned long adwItem;
    	unsigned long adwProbability;
    	int anNum;
    	int	anSpan;
    	int anAbilityOption;
    	unsigned char anFlag;
    };
    
    struct internalPackBoxItem
    {
    	unsigned long adwItem;
    	int anNum;
    	int anAbilityOption;
    	unsigned char anFlag;
    };
    
    union boxItems
    {
    	internalGiftBoxItem gb;
    	internalPackBoxItem pb;
    };
    
    typedef	struct _GIFTBOX
    {
    	unsigned long dwGiftbox;
    	unsigned long nSum;
    
    	std::vector<boxItems> itemList;
    	_GIFTBOX(): dwGiftbox(0), nSum(0) { }
    	[[nodiscard]] size_t size() const { return itemList.size(); }
    }	GIFTBOX, * PGIFTBOX;
    
    typedef struct _GIFTBOXRESULT
    {
    	unsigned long dwItem;
    	int nNum;
    	int nSpan;
    	int	nAbilityOption;
    	unsigned char nFlag;
    	
    	_GIFTBOXRESULT(): dwItem(0), nNum(0), nSpan(0), nAbilityOption(0), nFlag(0) { }
    }	GIFTBOXRESULT, * PGIFTBOXRESULT;
    
    
    class CGiftboxMan
    {
    		std::map<unsigned long, _GIFTBOX> itemList;
    
    	public:
    		CGiftboxMan();
    		~CGiftboxMan();
    
    		CGiftboxMan(CGiftboxMan const&) = delete;
    		CGiftboxMan& operator =(CGiftboxMan const&) = delete;
    		CGiftboxMan(CGiftboxMan&&) = delete;
    		CGiftboxMan& operator=(CGiftboxMan&&) = delete;
    	
    		static CGiftboxMan* GetInstance();
    		std::vector<boxItems>* getInternalItemList(unsigned long mainId);
    
    		bool isGiftbox(unsigned long itemId);
    		_GIFTBOX* getGiftbox(unsigned long itemId);
    
    		std::pair<std::map<unsigned long, _GIFTBOX>::iterator, bool> insert(unsigned long id, _GIFTBOX gb);
    	
    #ifdef __WORLDSERVER
    		bool open(unsigned long itemId, PGIFTBOXRESULT pGiftboxResult);
    #endif
    };
    
    typedef	struct _PACKITEMELEM
    {
    	DWORD dwPackItem;
    	unsigned long span;
    	std::vector<boxItems> itemList;
    	_PACKITEMELEM() : dwPackItem(0), span(0) {}
    	[[nodiscard]] size_t size() const { return itemList.size(); };
    } PACKITEMELEM, * PPACKITEMELEM;
    
    class CPackItem
    {
    		std::map<unsigned long, _PACKITEMELEM> itemList;
    
    	public:
    		CPackItem();
    		~CPackItem() = default;
    
    		CPackItem(CPackItem const&) = delete;
    		CPackItem& operator =(CPackItem const&) = delete;
    		CPackItem(CPackItem&&) = delete;
    		CPackItem& operator=(CPackItem&&) = delete;
    	
    		std::map<unsigned long, _PACKITEMELEM>& getItemList() { return itemList; }
    		static CPackItem* GetInstance();
    		_PACKITEMELEM Open(DWORD dwPackItem);
    
    		bool isPackItem(unsigned long itemId);
    		_PACKITEMELEM* getPackItem(unsigned long itemId);
    
    };
    #else
    #ifdef __WORLDSERVER
    #define	MAX_GIFTBOX_ITEM	128
    #ifndef __STL_GIFTBOX_VECTOR
    #define	MAX_GIFTBOX			256
    #endif // __STL_GIFTBOX_VECTOR
    typedef	struct	_GIFTBOX
    {
    	DWORD	dwGiftbox;
    	int		nSum;
    	int		nSize;
    	DWORD	adwItem[MAX_GIFTBOX_ITEM];
    	DWORD	adwProbability[MAX_GIFTBOX_ITEM];
    	int		anNum[MAX_GIFTBOX_ITEM];
    	BYTE	anFlag[MAX_GIFTBOX_ITEM];
    	int		anSpan[MAX_GIFTBOX_ITEM];
    	int		anAbilityOption[MAX_GIFTBOX_ITEM];
    	/*
    #ifdef __GIFTBOX0213
    	BOOL	bGlobal;
    	int		anTotal[MAX_GIFTBOX_ITEM];
    #endif	// __GIFTBOX0213
    	*/
    }	GIFTBOX,	*PGIFTBOX;
    
    typedef struct	_GIFTBOXRESULT
    {
    	DWORD	dwItem;
    	int		nNum;
    	BYTE nFlag;
    	int nSpan;
    	int	nAbilityOption;
    	_GIFTBOXRESULT()
    		{
    			dwItem	= 0;
    			nNum	= 0;
    			nFlag	= 0;
    			nSpan	= 0;
    			nAbilityOption	= 0;
    		}
    }	GIFTBOXRESULT,	*PGIFTBOXRESULT;
    
    /*
    #ifdef __GIFTBOX0213
    class CItemElem;
    class CDPMng;
    #endif	// __GIFTBOX0213
    */
    class	CGiftboxMan
    {
    private:
    #ifdef __STL_GIFTBOX_VECTOR
    	vector<GIFTBOX> m_vGiftBox;
    #else // __STL_GIFTBOX_VECTOR
    	int		m_nSize;
    	GIFTBOX	m_giftbox[MAX_GIFTBOX];
    #endif // __STL_GIFTBOX_VECTOR
    	map<DWORD, int>	m_mapIdx;
    	int	m_nQuery;
    
    public:
    	CGiftboxMan();
    	virtual	~CGiftboxMan()	{}
    	
    	static	CGiftboxMan* GetInstance( void );
    
    	/*
    #ifdef __GIFTBOX0213
    	BOOL	AddItem( DWORD dwGiftbox, DWORD dwItem, DWORD dwProbability, int nNum, BYTE nFlag = 0, int nTotal = 0     );
    	void	Upload( CDPMng* pdp );
    	void	Query( CDPMng* pdp, DWORD dwGiftbox, DWORD dwItem, int nNum, u_long idPlayer, CItemElem* pItemElem );
    	void	Restore( CDPMng* pdp, DWORD dwGiftbox, DWORD dwItem );
    	BOOL	Open( DWORD dwGiftbox, LPDWORD pdwItem, int* pnNum, u_long idPlayer, CItemElem* pItemElem );
    	BOOL	OpenLowest( DWORD dwGiftbox, LPDWORD pdwItem, int* pnNum );
    #else	// __GIFTBOX0213
    	*/
    	BOOL	AddItem( DWORD dwGiftbox, DWORD dwItem, DWORD dwProbability, int nNum, BYTE nFlag = 0, int nSpan	=     0, int nAbilityOption = 0 );
    	BOOL	Open( DWORD dwGiftBox, PGIFTBOXRESULT pGiftboxResult );
    //#endif	// __GIFTBOX0213
    	void	Verify( void );
    };
    #endif	// __WORLDSERVER
    
    #define	MAX_ITEM_PER_PACK	16
    #define	MAX_PACKITEM		512
    
    typedef	struct	_PACKITEMELEM
    {
    	DWORD dwPackItem;
    	int		nSize;
    	int		nSpan;
    	DWORD	adwItem[MAX_ITEM_PER_PACK];
    	int		anAbilityOption[MAX_ITEM_PER_PACK];
    	int		anNum[MAX_ITEM_PER_PACK];
    }	PACKITEMELEM,	*PPACKITEMELEM;
    
    class CPackItem
    {
    private:
    	int		m_nSize;
    	PACKITEMELEM	m_packitem[MAX_PACKITEM];
    	map<DWORD, int>	m_mapIdx;
    
    public:
    	CPackItem();
    	virtual	~CPackItem()	{}
    
    	static	CPackItem*	GetInstance( void );
    
    	BOOL	AddItem( DWORD dwPackItem, DWORD dwItem, int nAbilityOption, int nNum );
    	PPACKITEMELEM	Open( DWORD dwPackItem );
    };
    #endif
    ```
    find and replace
    ```CPP
    #ifdef __BoxPreview
    	BOOL			LoadGiftbox(const char* lpszFileName) const;
    	BOOL			LoadPackItem(const char* lpszFileName) const;
    #else
    	BOOL			LoadGiftbox( LPCTSTR lpszFileName );
    	BOOL			LoadPackItem( LPCTSTR lpszFileName );
    #endif
    ```
- **WndItemCtrl.cpp**
    find and replace inside of _CWndItemCtrl::OnLButtonDblClk_
    ```CPP
    #if __VER >= 8 //__CSC_VER8_5
    	if( IsUsableItem( pItemElem ) || (m_useDieFlag && pItemElem != NULL))
    #else
    	if( IsUsableItem( pItemElem ) )
    #endif //__CSC_VER8_5
    	{
    		if( IsSelectedItem( nItem ) == FALSE )
    		{
    			m_dwSelectAry.RemoveAll();
    			m_dwSelectAry.Add( nItem );
    			m_nCurSelect = nItem;
    			m_pFocusItem = pItemElem;
    		}
    #ifdef __BoxPreview
    		if (CGiftboxMan::GetInstance()->isGiftbox(pItemElem->m_dwItemId) ||  CPackItem::GetInstance()->isPackItem(pItemElem->m_dwItemId))
    		{
    			if (g_WndMng.IsOpenWnd(APP_BOXPREVIEW))
    			{
    				g_WndMng.wndBoxPreview->setData(pItemElem->m_dwItemId, pItemElem->m_dwObjId, APP_INVENTORY);
    				return;
    			}
    			g_WndMng.wndBoxPreview = new CWndBoxPreview();
    			g_WndMng.wndBoxPreview->Initialize();
    			g_WndMng.wndBoxPreview->setData(pItemElem->m_dwItemId, pItemElem->m_dwObjId, APP_INVENTORY);
    			return;
    		}
    #endif
    
    		CWndBase* pParent = (CWndBase*)GetParentWnd();
    		pParent->OnChildNotify( WIN_DBLCLK,m_nIdWnd,(LRESULT*)m_pFocusItem); 
    ```
- **WndManager.cpp**
    bottom of _CWndMgr::CWndMgr()_
    ```CPP
    #ifdef __BoxPreview
	    wndBoxPreview = nullptr;
    #endif
    ```
    bottom of _CWndMgr::Free()_
    ```CPP
    #ifdef __BoxPreview
    	SAFE_DELETE(wndBoxPreview);
    #endif
    ```
    bottom of _CWndMgr::OnDestroyChildWnd_
    ```CPP
    #ifdef __BoxPreview
    	if (wndBoxPreview == pWndChild)
    	{
    		SAFE_DELETE(wndBoxPreview);
    		pWndChild = nullptr;
    	}
    #endif
    ```
- **WndManager.h**
    in the class under public scope
    ```CPP
    #ifdef __BoxPreview
    	CWndBoxPreview* wndBoxPreview;
    #endif
    ```
- **WndNeuz.cpp**
    find and add in function _CWndNeuz::CreateControl_
    ```CPP
    	case WTYPE_LISTBOX:
    #ifdef __BoxPreview
    		if (GetWndId() == APP_BOXPREVIEW)
    			pWndBase = new boxPreviewListBox;
    		else
    #endif
    			pWndBase = new CWndListBox;
    		((CWndListBox*)pWndBase)->Create( dwWndStyle, lpWndCtrl->rect, this, lpWndCtrl->dwWndId );
    ```
- **WndBoxPreview.h** _(Make and add to interface and neuz project)_
    ```CPP
    #pragma once
    #include "WndNeuz.h"
    #include "WndControl.h"
    
    #ifdef __BoxPreview
    class boxPreviewListBox final : public CWndListBox
    {
    	public:
    		void OnDraw(C2DRender* p2DRender) override;
    };
    
    class CWndBoxPreview final : public CWndNeuz
    {
    		enum class sortByType { alphabetical, quantity, percentage };
    		sortByType sortType = sortByType::alphabetical;
    		bool ascending, isGiftBox;
    		unsigned long tickTimer;
    
    		CItemElem* itemElem;
    		CTexture* itemTexture;
    		CWndStatic* renderLocationStatic, *itemNameStatic;
    		CWndButton* sortNameButton, *sortNumberButton, *sortPercentButton, *okButton;
    		CWndListBox* itemListBox;
    		void* objectPtr;
    
    	public:
    		CWndBoxPreview();
    		~CWndBoxPreview();
    
    		CWndBoxPreview(CWndBoxPreview const&) = delete;
    		CWndBoxPreview& operator =(CWndBoxPreview const&) = delete;
    		CWndBoxPreview(CWndBoxPreview&&) = delete;
    		CWndBoxPreview& operator=(CWndBoxPreview&&) = delete;
    
    		[[nodiscard]] std::vector<boxItems>* getBoxItems() const;
    		[[nodiscard]] float getMaxNumberForPercent() const;
    		[[nodiscard]] bool IsGiftbox() const;
    
    		void setData(unsigned long id, unsigned char inventorySlot, int type);
    		void setDataProp(unsigned long id, int type);
    		void sort(CWndBoxPreview::sortByType st);
    		bool updateObjectPtr(unsigned long itemId);
    
    		BOOL OnDropIcon(LPSHORTCUT pShortcut, CPoint point) override;
    		void OnLButtonUp(UINT nFlags, CPoint point) override;
    		BOOL Initialize(CWndBase* pWndParent = nullptr, DWORD = 0) override;
    		BOOL OnChildNotify(UINT message, UINT nID, LRESULT* pLResult) override;
    		void OnDraw(C2DRender* p2DRender) override;
    		void OnInitialUpdate() override;
    };
    #endif
    ```
- **WndBoxPreview.cpp** _(Make and add to interface and neuz project)_
    ```CPP
    #include "StdAfx.h"
    #include "WndBoxPreview.h"
    #include "ResData.h"
    #include "DPClient.h"
    
    extern CDPClient g_DPlay;
    
    
    #ifdef __BoxPreview
    void boxPreviewListBox::OnDraw(C2DRender* p2DRender)
    {
    	CWndBoxPreview* boxPreview = dynamic_cast<CWndBoxPreview*>(GetParentWnd());
    	
    	std::vector<boxItems>* ptr = boxPreview->getBoxItems();
    	if (ptr == nullptr)
    		return;
    
    	m_nFontHeight = 34;
    	const int nPage = GetClientRect().Height() / m_nFontHeight;
    	const int nRange = ptr->size();
    	if (IsWndStyle(WBS_VSCROLL))
    	{
    		m_wndScrollBar.SetVisible(true);
    		m_wndScrollBar.SetScrollRange(0, nRange);
    		m_wndScrollBar.SetScrollPage(nPage);
    	}
    	else
    		m_wndScrollBar.SetVisible(false);
    
    	const unsigned long width = GetClientRect().Width();
    	CPoint pt(0, 1);
    	for (int i = m_wndScrollBar.GetScrollPos(); i < nRange; ++i)
    	{
    		if (i > static_cast<int>(nPage + m_wndScrollBar.GetScrollPos()))
    			break;
    		
    		boxItems& bi = ptr->at(i);
    		const ItemProp* const iProp = prj.GetItemProp(static_cast<int>(boxPreview->IsGiftbox() ? bi.gb.adwItem :     bi.pb.adwItem));
    		if (iProp)
    		{
    			CTexture* itemTexture = CWndBase::m_textureMng.AddTexture(g_Neuz.m_pd3dDevice, MakePath(DIR_ITEM,     iProp->szIcon), 0xffff00ff);
    			itemTexture->Render(p2DRender, pt);
    
    			CRect rectTT = CRect(0, pt.y, width, pt.y + m_nFontHeight - 1);
    			if (rectTT.PtInRect(m_ptMouse))
    			{
    				CItemElem elem;
    				CPoint pt2 = m_ptMouse;
    				ClientToScreen(&pt2);
    				ClientToScreen(&rectTT);
    				elem.m_dwItemId = iProp->dwID;
    				elem.SetAbilityOption(boxPreview->IsGiftbox() ? bi.gb.anAbilityOption : bi.pb.anAbilityOption);
    				elem.SetFlag(boxPreview->IsGiftbox() ? bi.gb.anFlag : bi.pb.anFlag);
    				g_WndMng.PutToolTip_Item(&elem, pt2, &rectTT);
    			}
    			
    #ifdef __UtilityFontFunctions
    			p2DRender->TextOut(32, pt.y + 5, 135, 20, iProp->szName, 0xFF000000);
    			p2DRender->TextOut(170, pt.y + 5, boxPreview->IsGiftbox() ? std::to_string(bi.gb.anNum).c_str() :     std::to_string(bi.pb.anNum).c_str(), 0xAA000000);
    			if (boxPreview->IsGiftbox())
    			{
    				std::string percentage(16, '\0');
    				const float calc = static_cast<float>(static_cast<float>(bi.gb.adwProbability) /     boxPreview->getMaxNumberForPercent()) * 100.0f;
    				const long amount = std::snprintf(&percentage[0], percentage.size(), "%.2f%%", calc);
    				if (amount)
    					percentage.resize(amount);
    				
    				p2DRender->TextOut(195, pt.y + 5, percentage.c_str(), 0xAA000000);
    			}
    
    #else
    			std::string tempStr = iProp->szName;
    			if (tempStr.length() > 9)
    				tempStr = tempStr.substr(0, 9);
    			tempStr += "..";
    			p2DRender->TextOut(33, pt.y + 3, tempStr.c_str(), 0xFF000000);
    			p2DRender->TextOut(135, pt.y + 3, boxPreview->IsGiftbox() ? std::to_string(bi.gb.anNum).c_str() :     std::to_string(bi.pb.anNum).c_str(), 0xAA000000);
    			if (boxPreview->IsGiftbox())
    				p2DRender->TextOut(33, pt.y + 15, std::to_string(bi.gb.adwProbability).c_str(), 0xAA000000);
    #endif
    		}
    		else
    			continue;
    
    		pt.y += m_nFontHeight;
    	}
    }
    
    
    
    CWndBoxPreview::CWndBoxPreview() : ascending(true), isGiftBox(false), tickTimer(0), itemElem(nullptr),     itemTexture(nullptr), renderLocationStatic(nullptr), itemNameStatic(nullptr), sortNameButton(nullptr),     sortNumberButton(nullptr),
    	sortPercentButton(nullptr), okButton(nullptr), itemListBox(nullptr), objectPtr(nullptr)
    {
    }
    
    void CWndBoxPreview::setDataProp(const unsigned long id, const int type)
    {
    	if (g_pPlayer && (updateObjectPtr(id)))
    	{
    		if (itemElem)
    		{
    			itemElem->SetExtra(0);
    			itemTexture = nullptr;
    			itemElem = nullptr;
    		}
    
    		ItemProp* iProp = prj.GetItemProp(id);
    		if (iProp)
    		{
    			itemTexture = CWndBase::m_textureMng.AddTexture(g_Neuz.m_pd3dDevice, MakePath(DIR_ITEM,     iProp->szIcon), 0xffff00ff);
    
    			if (itemNameStatic)
    				itemNameStatic->SetTitle(iProp->szName);
    		}
    
    	}
    }
    
    void CWndBoxPreview::setData(const unsigned long id, const unsigned char inventorySlot, const int type) 
    {
    	if (g_pPlayer && (updateObjectPtr(id)))
    	{
    		if (itemElem)
    		{
    			itemElem->SetExtra(0);
    			itemTexture = nullptr;
    			itemElem = nullptr;
    		}
    		if (type == APP_INVENTORY)
    		{		
    			itemElem = g_pPlayer->m_Inventory.GetAtId(inventorySlot);
    			itemElem->SetExtra(itemElem->m_nItemNum);
    			ItemProp* iProp = itemElem->GetProp();
    			if (iProp)
    				itemTexture = CWndBase::m_textureMng.AddTexture(g_Neuz.m_pd3dDevice, MakePath(DIR_ITEM,     iProp->szIcon), 0xffff00ff);
    
    			if (itemNameStatic)
    				itemNameStatic->SetTitle(itemElem->GetName());
    		}
    		else
    		{
    			
    		}// nothing yet
    		sort(sortType);
    	}
    }
    
    CWndBoxPreview::~CWndBoxPreview()
    {
    	if (itemElem)
    		itemElem->SetExtra(0);
    	itemElem = nullptr;
    	itemTexture = nullptr;
    	objectPtr = nullptr;
    }
    
    
    BOOL CWndBoxPreview::Initialize(CWndBase* pWndParent, DWORD)
    {
    	return InitDialog(g_Neuz.GetSafeHwnd(), APP_BOXPREVIEW, 0, CPoint(0, 0), pWndParent);
    }
    
    void CWndBoxPreview::OnInitialUpdate()
    {
    	CWndNeuz::OnInitialUpdate();
    	renderLocationStatic = dynamic_cast<CWndStatic*>(GetDlgItem(WIDC_STATIC1));
    	itemNameStatic = dynamic_cast<CWndStatic*>(GetDlgItem(WIDC_STATIC2));
    	
    	sortNameButton = dynamic_cast<CWndButton*>(GetDlgItem(WIDC_BUTTON1));
    	sortNumberButton = dynamic_cast<CWndButton*>(GetDlgItem(WIDC_BUTTON2));
    	sortPercentButton = dynamic_cast<CWndButton*>(GetDlgItem(WIDC_BUTTON3));
    	okButton = dynamic_cast<CWndButton*>(GetDlgItem(WIDC_BUTTON10));
    
    	itemListBox = dynamic_cast<CWndListBox*>(GetDlgItem(WIDC_LISTBOX1));
    	
    	MoveParentCenter();
    
    }
    
    void CWndBoxPreview::OnDraw(C2DRender* p2DRender)
    {
    	if (itemElem && itemTexture)
    	{
    		itemTexture->Render(p2DRender, CPoint(renderLocationStatic->GetWndRect().left,     renderLocationStatic->GetWndRect().top));
    		p2DRender->TextOut(renderLocationStatic->GetWndRect().left, renderLocationStatic->GetWndRect().bottom - 6,     0.8f, 0.8f, std::to_string(itemElem->GetExtra()).c_str(), 0xff000000);
    	}
    	
    	if (!itemElem)
    		okButton->EnableWindow(false);
    	else
    	{
    		if (tickTimer)
    		{
    			if (g_tmCurrent > tickTimer)
    			{
    				okButton->EnableWindow(true);
    				tickTimer = 0;
    			}
    		}
    		else
    			okButton->EnableWindow(true);
    	}
    }
    
    bool CWndBoxPreview::updateObjectPtr(const unsigned long itemId)
    {
    	if (itemListBox)
    	{
    		int itemListSize = 0;
    
    		_GIFTBOX* giftbox = CGiftboxMan::GetInstance()->getGiftbox(itemId);
    		if (giftbox)
    		{
    			itemListSize = giftbox->itemList.size();
    			objectPtr = static_cast<void*>(giftbox);
    			isGiftBox = true;
    		}
    		
    		_PACKITEMELEM* packBox = CPackItem::GetInstance()->getPackItem(itemId);
    		if (packBox)
    		{
    			itemListSize = packBox->itemList.size();
    			objectPtr = static_cast<void*>(packBox);
    			isGiftBox = false;
    		}
    		itemListBox->ResetContent();
    		for (int i = 0; i < itemListSize; ++i)
    			itemListBox->AddString("");
    
    		return true;
    	}
    	return false;
    }
    
    
    BOOL CWndBoxPreview::OnDropIcon(LPSHORTCUT pShortcut, CPoint point)
    {
    	if (tickTimer > 0)
    		return false;
    
    	CWndBase* pWndFrame = pShortcut->m_pFromWnd->GetFrameWnd();
    	if (!pWndFrame)
    		return false;
    
    	if (pWndFrame->GetWndId() != APP_INVENTORY && pWndFrame->GetWndId() != APP_BAG_EX)
    	{
    		SetForbid(true);
    		return false;
    	}
    	
    	if (pShortcut->m_dwShortcut == SHORTCUT_ITEM)
    	{
    		if (renderLocationStatic)
    		{
    			if (GetClientRect().PtInRect(point)) // anywhere on app
    			{
    				CItemElem* pTempElem = dynamic_cast<CItemElem*>(g_pPlayer->GetItemId(pShortcut->m_dwId));
    				if (!pTempElem)
    					return false;
    
    				ItemProp* pItemProp = pTempElem->GetProp();
    				if (!pItemProp)
    					return false;
    
    				if (updateObjectPtr(pTempElem->m_dwItemId))
    				{
    					itemElem = pTempElem;
    					itemElem->SetExtra(itemElem->m_nItemNum);
    					itemTexture = CWndBase::m_textureMng.AddTexture(g_Neuz.m_pd3dDevice, MakePath(DIR_ITEM,     pItemProp->szIcon), 0xffff00ff);
    
    					if (itemNameStatic)
    						itemNameStatic->SetTitle(itemElem->GetName());
    
    					sort(sortType);
    					return true;
    				}
    			}
    		}
    	}
    	return false;
    }
    
    void CWndBoxPreview::OnLButtonUp(UINT nFlags, CPoint point)
    {
    	if (renderLocationStatic && renderLocationStatic->GetWndRect().PtInRect(point))
    	{
    		if (itemElem)
    		{
    			itemElem->SetExtra(0);
    			itemElem = nullptr;
    			itemTexture = nullptr;
    			if (itemNameStatic)
    				itemNameStatic->SetTitle("");
    
    			objectPtr = nullptr;
    		}
    	}
    }
    
    float CWndBoxPreview::getMaxNumberForPercent() const
    {
    	return isGiftBox ? static_cast<float>(static_cast<_GIFTBOX*>(objectPtr)->nSum) : 0.f;
    }
    
    bool CWndBoxPreview::IsGiftbox() const
    {
    	return isGiftBox;
    }
    
    std::vector<boxItems>* CWndBoxPreview::getBoxItems() const
    {
    	return objectPtr ? isGiftBox ? &static_cast<_GIFTBOX*>(objectPtr)->itemList :     &static_cast<_PACKITEMELEM*>(objectPtr)->itemList : nullptr;
    };
    
    void CWndBoxPreview::sort(const sortByType st)
    {
    	if (objectPtr && itemElem)
    	{
    		if (sortType == st)
    			ascending = !ascending;
    		else
    			sortType = st;
    
    		std::vector<boxItems>::iterator begin, end;
    		if (isGiftBox)
    			begin = static_cast<_GIFTBOX*>(objectPtr)->itemList.begin(), end =     static_cast<_GIFTBOX*>(objectPtr)->itemList.end();
    		else
    			begin = static_cast<_PACKITEMELEM*>(objectPtr)->itemList.begin(), end =     static_cast<_PACKITEMELEM*>(objectPtr)->itemList.end();
    
    		switch (st)
    		{
    			case sortByType::alphabetical:
    				std::sort(begin, end, [&](const boxItems& a, const boxItems& b) -> bool {
    						ItemProp* iaProp, *ibProp;
    						if (isGiftBox)
    							iaProp = prj.GetItemProp(static_cast<int>(a.gb.adwItem)), ibProp =     prj.GetItemProp(static_cast<int>(b.gb.adwItem));
    						else
    							iaProp = prj.GetItemProp(static_cast<int>(a.pb.adwItem)), ibProp =     prj.GetItemProp(static_cast<int>(b.pb.adwItem));
    
    						if (iaProp && ibProp)
    							return ascending ? std::string(iaProp->szName) < std::string(ibProp->szName) :     std::string(iaProp->szName) > std::string(ibProp->szName); // lazy std::string
    						return false;
    					});
    				break;
    			case sortByType::quantity:
    				std::sort(begin, end, [&](const boxItems& a, const boxItems& b) -> bool {
    						return ascending ? 
    							isGiftBox ? a.gb.anNum < b.gb.anNum : a.pb.anNum < b.pb.anNum
    							: isGiftBox ? a.gb.anNum > b.gb.anNum : a.pb.anNum > b.pb.anNum;
    					});							
    				break;
    			case sortByType::percentage:
    				if (isGiftBox)
    					std::sort(begin, end, [&](const boxItems& a, const boxItems& b) -> bool {
    							return ascending
    								? static_cast<float>(a.gb.adwProbability) /     static_cast<float>(static_cast<_GIFTBOX*>(objectPtr)->nSum) <     static_cast<float>(b.gb.adwProbability) /     static_cast<float>(static_cast<_GIFTBOX*>(objectPtr)->nSum)
    									: static_cast<float>(a.gb.adwProbability) /     static_cast<float>(static_cast<_GIFTBOX*>(objectPtr)->nSum) >     static_cast<float>(b.gb.adwProbability) /     static_cast<float>(static_cast<_GIFTBOX*>(objectPtr)->nSum);
    						});
    				break;
    			default: break;
    		}
    	}	
    }
    
    BOOL CWndBoxPreview::OnChildNotify(const UINT message, const UINT nID, LRESULT* pLResult)
    {
    	switch (nID)
    	{
    		case WIDC_BUTTON1:
    		{
    			sort(sortByType::alphabetical);
    			break;
    		}
    		case WIDC_BUTTON2:
    		{
    			sort(sortByType::quantity);
    			break;
    		}
    		case WIDC_BUTTON3:
    		{	
    			sort(sortByType::percentage);
    			break;
    		}
    		case WIDC_BUTTON10:
    		{
    			if (itemElem != nullptr)
    			{
    				okButton->EnableWindow(false);
    				tickTimer = g_tmCurrent + 500;
    				g_DPlay.SendDoUseItem(MAKELONG(ITYPE_ITEM, itemElem->m_dwObjId), NULL_ID, -1, FALSE);
    				if (itemElem->m_nItemNum > 1)
    					itemElem->SetExtra(itemElem->GetExtra() - 1);
    				else
    				{
    					itemElem->SetExtra(0);
    					itemElem = nullptr;
    					itemListBox->ResetContent();
    					objectPtr = nullptr;
    					itemNameStatic->SetTitle("");
    				}
    			}
    			else
    				g_WndMng.PutString("Needs to be a valid Item");
    			break;
    		}
    		default: break;
    	}
    
    	return CWndNeuz::OnChildNotify(message, nID, pLResult);
    }
    #endif
    ```

## Utility Font Functions
- That can be [found here on RageZone](http://forum.ragezone.com/f457/limit-text-1192086/)

## Resource Edits
---
- **ResData.h**
    ```
    #define APP_BOXPREVIEW 2052
    ```
- **Resdata.inc**
    ```
    APP_BOXPREVIEW "WndTile00.tga" "" 1 288 384 0x2410000 26
    {
    	IDS_RESDATA_INC_009700
    }
    {
    	IDS_RESDATA_INC_009700
    }
    {
        WTYPE_STATIC WIDC_STATIC1 "WndChgElemItem.bmp" 0 20 20 51 51 0x2220002 0 0 0 0 46 112 169
        {
    		IDS_RESDATA_INC_000001
        }
        {
    		IDS_RESDATA_INC_000001
        }
        WTYPE_STATIC WIDC_STATIC2 "" 0 58 26 216 43 0x2220010 0 0 0 0 46 112 169
        {
    		IDS_RESDATA_INC_000001
        }
        {
    		IDS_RESDATA_INC_000001
        }
        WTYPE_LISTBOX WIDC_LISTBOX1 "WndEditTile00.tga" 1 8 94 280 300 0x20020000 0 0 0 0 46 112 169
        {
    		IDS_RESDATA_INC_000001
        }
        {
    		IDS_RESDATA_INC_000001
        }
        WTYPE_BUTTON WIDC_BUTTON10 "ButtNormal03.bmp" 1 42 318 242 342 0x220000 0 0 0 0 0 0 0
        {
    		IDS_RESDATA_INC_009704
        }
        {
    		IDS_RESDATA_INC_000001
        }
        WTYPE_BUTTON WIDC_BUTTON1 "ButtNormal01.bmp" 1 8 68 112 92 0x220000 0 0 0 0 0 0 0
        {
    		IDS_RESDATA_INC_009701
        }
        {
    		IDS_RESDATA_INC_000001
        }
        WTYPE_BUTTON WIDC_BUTTON2 "ButtNormal00.bmp" 1 116 68 188 92 0x220000 0 0 0 0 0 0 0
        {
    		IDS_RESDATA_INC_009702
        }
        {
    		IDS_RESDATA_INC_000001
        }
        WTYPE_BUTTON WIDC_BUTTON3 "ButtNormal00.bmp" 1 192 68 276 92 0x220000 0 0 0 0 0 0 0
        {
    		IDS_RESDATA_INC_009703
        }
        {
    		IDS_RESDATA_INC_000001
        }
    }
    ```
- **Resdata.txt.txt**
    ```
    IDS_RESDATA_INC_009700 Box Opener
    IDS_RESDATA_INC_009701 Item Name
    IDS_RESDATA_INC_009702 Quantity
    IDS_RESDATA_INC_009703 Percent
    IDS_RESDATA_INC_009704 Open Box
    ```

## Changes based on VS
---
_if you run an older compiler_
- Remove [[nodiscard]] (_This is used in later compiler versions so if it errors for you_)

## License
---
_the work of Kia#0001_

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)

