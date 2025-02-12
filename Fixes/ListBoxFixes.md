## List Box Optimization
---
The original code runs the PaintListBox function. You can actually read what it does, but it starts to "render" all elements, even if they are off the screen. This is the infamous lag of using the AdminItemCreator.
- **CWndListBox::OnDraw**
	```cpp
		m_nFontHeight = GetFontHeight() + m_nLineSpace;
		{
	#ifndef __fixes
			CPoint pt(3, 3);
			pt.y -= m_nFontHeight * m_wndScrollBar.GetScrollPos();

			PaintListBox(p2DRender, pt, m_listItemArray);
			int nPage = GetClientRect().Height() / m_nFontHeight;
			int nRange = m_listItemArray.GetSize();

			if (IsWndStyle(WBS_VSCROLL))
			{
				m_wndScrollBar.SetVisible(TRUE);
				m_wndScrollBar.SetScrollRange(0, nRange);
				m_wndScrollBar.SetScrollPage(nPage);
			}
			else
				m_wndScrollBar.SetVisible(FALSE);
	#else
			const int nPage = GetClientRect().Height() / m_nFontHeight;
			const int nRange = m_listItemArray.GetSize();

			if (IsWndStyle(WBS_VSCROLL))
			{
				m_wndScrollBar.SetVisible(TRUE);
				m_wndScrollBar.SetScrollRange(0, nRange);
				m_wndScrollBar.SetScrollPage(nPage);
			}
			else
				m_wndScrollBar.SetVisible(FALSE);

			const int nScrollBarWidth = IsWndStyle(WBS_VSCROLL) ? m_wndScrollBar.GetClientRect().Width() : 0;
			CPoint pt(3, 3);
			for (int i = m_wndScrollBar.GetScrollPos(); i < static_cast<unsigned>(m_listItemArray.GetSize()); i++)
			{
				if (i > nPage + m_wndScrollBar.GetScrollPos())
					break;

				tagITEM* pListItem = static_cast<tagITEM*>(m_listItemArray.GetAt(i));
				if (!pListItem->m_bIsVisible)
					continue;

				if (pListItem->m_bIsValid)
				{
					pListItem->m_rect.left = pt.x;
					pListItem->m_rect.top = pt.y;
					pListItem->m_rect.right = pt.x + m_rectWindow.Width() - nScrollBarWidth;
					pListItem->m_rect.bottom = pt.y + m_nFontHeight;

					if (i == m_nCurSelect)
						pListItem->m_strWord.SetColor(m_nSelectColor);
					else
					{
						CPoint point = GetMousePoint();
						if (pListItem->m_rect.PtInRect(point))
							pListItem->m_strWord.SetColor(m_dwOnMouseColor);
						else
							pListItem->m_strWord.SetColor(m_nFontColor);
					}
				}
				else
					pListItem->m_strWord.SetColor(m_dwInvalidColor);
				p2DRender->TextOut_EditString(static_cast<int>(m_nLeftMargin + pt.x), pt.y, pListItem->m_strWord);
				pt.y += m_nFontHeight;
			}
	#endif
	```


## Small Optimzation with Inventories
This is also included because it follows the same as above -- when you implement 168 Inventory slots, you end up rendering all the items.
- **CWndItemCtrl::OnDraw**  
	_either undefine or remove_
	```cpp
	#ifndef __fixes
		int nRange = m_pItemContainer->m_dwIndexNum / nWidth;// - nPage;
		if( m_pItemContainer->m_dwIndexNum % nWidth )
			nRange++;
		m_wndScrollBar.SetScrollRange( 0, nRange );
		m_wndScrollBar.SetScrollPage( nPage );

		pt.y = 0;
		pt.y += m_wndScrollBar.GetScrollPos() * nWidth;
	#endif
	```
	_Add with or without ifdefs_
	```cpp
		if( nParent == APP_INVENTORY || nParent == APP_BAG_EX)
		{
	#ifdef __fixes
			int nRange = m_pItemContainer->m_dwIndexNum / nWidth;// - nPage;
			if (m_pItemContainer->m_dwIndexNum % nWidth)
				nRange++;
			m_wndScrollBar.SetScrollRange(0, nRange);
			m_wndScrollBar.SetScrollPage(nPage);
			pt.y = 0;
			pt.y += m_wndScrollBar.GetScrollPos() * nWidth;
			const int maxLoop = pt.y + (nPage * nWidth + nWidth);
	#endif

			for( int i = pt.y; i < m_pItemContainer->GetSize(); i++ ) 
			{
				if( i < 0 ) continue;
	#ifdef __fixes
				if (i > maxLoop)
					break;
	#endif
	```
	_same_
	```cpp
	#ifdef __fixe
			int nRange = m_nArrayCount / nWidth;// - nPage;
			if (m_nArrayCount % nWidth)
				nRange++;
			
			m_wndScrollBar.SetScrollRange(0, nRange);
			m_wndScrollBar.SetScrollPage(nPage);
			
			pt.y = 0;
			pt.y += m_wndScrollBar.GetScrollPos() * nWidth;

			const int maxLoop = pt.y + (nPage * nWidth + nWidth);
			for (int i = pt.y; i < m_nArrayCount; i++)
	#else
			for( int i = 0; i < m_nArrayCount; i++ )
	#endif
			{
	#ifdef __fixes
				if (i < 0 || i > maxLoop)
					break;
	#endif
	```
