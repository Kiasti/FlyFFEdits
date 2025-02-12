

## Limit X and Y for Text (2drender)
Before
![Before](https://i.imgur.com/UJAaelK.png) 
After (I forgot that the ? renders so I miscalculated the limits)
![After](https://i.imgur.com/Bf1yDBX.png)


_Automatically scales down font based on restrictions set for x and y._
- **2dRender.cpp**
  ```CPP
    void C2DRender::TextOut(const int x, const int y, const int limitX, const int limitY, const LPCTSTR pszString, const unsigned long dwColor, const unsigned long dwShadowColor)
    {
        if (m_pFont)
        {
            const SIZE size = m_pFont->GetTextExtent(pszString);
            float scale = 1.0f;
            if (size.cx > limitX || size.cy > limitY)
            {
                const float sizeX = static_cast<float>(limitX) / static_cast<float>(size.cx);
                const float sizeY = static_cast<float>(limitY) / static_cast<float>(size.cy);
                scale = sizeX < sizeY ? sizeX : sizeY;
            }
            const CRect rect(x + m_ptOrigin.x, y + m_ptOrigin.y, x + m_ptOrigin.x + size.cx, y + m_ptOrigin.y + size.cy);

            if (m_clipRect.RectLapRect(rect))
            {
                if (dwShadowColor & 0xff000000)m_pFont->DrawText(static_cast<FLOAT>(x + m_ptOrigin.x + 1), static_cast<FLOAT>(y + m_ptOrigin.y), scale, scale, dwShadowColor, const_cast<TCHAR*>(pszString));
                m_pFont->DrawText(static_cast<FLOAT>(x + m_ptOrigin.x), static_cast<FLOAT>(y + m_ptOrigin.y), scale, scale, dwColor, const_cast<TCHAR*>(pszString));
            }
        }
    }
  ```
- **2dRender.h**
  ```CPP
    void TextOut(int x, int y, int limitX, int limitY, LPCTSTR pszString, unsigned long dwColor, unsigned long dwShadowColor = 0x00000000);
  ```
  