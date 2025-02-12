
```cpp
inline D3DXVECTOR3 CreateTexVec(const float a, const float b)
{
    return D3DXVECTOR3(a - 0.5f, b - 0.5f, 0.0f);
}
BOOL C2DRender::RenderTextureForMinimap(const CRect& kRect, CTexture* pTexture, CTexture* pAlphaTex, const CSize& kSize)
{
    if (!pTexture || !pAlphaTex)
        return FALSE;

    static CTexture kTexFrame;
    if (!kTexFrame.m_pTexture)
        kTexFrame.LoadTexture(m_pd3dDevice, MakePath(DIR_THEME, "map.tga"), 0xffff00ff, TRUE);

    const float fScaleX = 1.0f;
    const float fScaleY = 1.0f;
    const CPoint pt = m_ptOrigin - pAlphaTex->m_ptCenter;
    const float left = static_cast<float>(pt.x);
    const float top = static_cast<float>(pt.y);
    const float right = left + (fScaleX * kSize.cx);
    const float bottom = top + (fScaleY * kSize.cy);

    const TEXTUREVERTEX vertex[4] {
        { CreateTexVec(left, top), 1.0f, pAlphaTex->m_fuLT, pAlphaTex->m_fvLT },
        { CreateTexVec(right, top), 1.0f, pAlphaTex->m_fuRT, pAlphaTex->m_fvRT },
        { CreateTexVec(left, bottom), 1.0f, pAlphaTex->m_fuLB, pAlphaTex->m_fvLB},
        { CreateTexVec(right, bottom),1.0f, pAlphaTex->m_fuRB, pAlphaTex->m_fvRB},
    };


    m_pd3dDevice->SetRenderState(D3DRS_CULLMODE, D3DCULL_NONE);
    m_pd3dDevice->SetRenderState(D3DRS_ALPHABLENDENABLE, true);

    m_pd3dDevice->SetTexture(0, pTexture->m_pTexture);
    m_pd3dDevice->SetTexture(1, pAlphaTex->m_pTexture);

    m_pd3dDevice->SetSamplerState(0, D3DSAMP_MINFILTER, D3DTEXF_LINEAR);
    m_pd3dDevice->SetSamplerState(0, D3DSAMP_MAGFILTER, D3DTEXF_LINEAR);
    m_pd3dDevice->SetSamplerState(1, D3DSAMP_MINFILTER, D3DTEXF_LINEAR);
    m_pd3dDevice->SetSamplerState(1, D3DSAMP_MAGFILTER, D3DTEXF_LINEAR);

    m_pd3dDevice->SetTextureStageState(0, D3DTSS_COLOROP, D3DTOP_SELECTARG1);
    m_pd3dDevice->SetTextureStageState(0, D3DTSS_COLORARG1, D3DTA_TEXTURE);
    m_pd3dDevice->SetTextureStageState(0, D3DTSS_ALPHAOP, D3DTOP_SELECTARG1);
    m_pd3dDevice->SetTextureStageState(0, D3DTSS_ALPHAARG1, D3DTA_TEXTURE);

    m_pd3dDevice->SetTextureStageState(1, D3DTSS_COLOROP, D3DTOP_SELECTARG1);
    m_pd3dDevice->SetTextureStageState(1, D3DTSS_COLORARG1, D3DTA_CURRENT);
    m_pd3dDevice->SetTextureStageState(1, D3DTSS_ALPHAOP, D3DTOP_MODULATE);
    m_pd3dDevice->SetTextureStageState(1, D3DTSS_ALPHAARG1, D3DTA_TEXTURE);
    m_pd3dDevice->SetTextureStageState(1, D3DTSS_TEXCOORDINDEX, 0);
    m_pd3dDevice->SetFVF(D3DFVF_TEXTUREVERTEX);
    m_pd3dDevice->DrawPrimitiveUP(D3DPT_TRIANGLESTRIP, 2, vertex, sizeof(TEXTUREVERTEX));

    m_pd3dDevice->SetTextureStageState(1, D3DTSS_COLOROP, D3DTOP_MODULATE);
    m_pd3dDevice->SetTextureStageState(1, D3DTSS_COLORARG1, D3DTA_TEXTURE);
    m_pd3dDevice->SetTextureStageState(1, D3DTSS_COLORARG2, D3DTA_DIFFUSE);
    m_pd3dDevice->SetTextureStageState(1, D3DTSS_ALPHAOP, D3DTOP_MODULATE);
    m_pd3dDevice->SetTextureStageState(1, D3DTSS_ALPHAARG2, D3DTA_CURRENT);
 
    m_pd3dDevice->SetTexture(0, kTexFrame.m_pTexture);
    m_pd3dDevice->SetTexture(1, nullptr); 
 
    m_pd3dDevice->SetTextureStageState(0, D3DTSS_COLOROP, D3DTOP_MODULATE);
    m_pd3dDevice->SetTextureStageState(0, D3DTSS_COLORARG1, D3DTA_TEXTURE);
    m_pd3dDevice->SetTextureStageState(0, D3DTSS_COLORARG2, D3DTA_DIFFUSE);
    m_pd3dDevice->SetTextureStageState(0, D3DTSS_ALPHAOP, D3DTOP_MODULATE);
    m_pd3dDevice->SetTextureStageState(0, D3DTSS_ALPHAARG1, D3DTA_TEXTURE);
    m_pd3dDevice->SetTextureStageState(0, D3DTSS_ALPHAARG2, D3DTA_CURRENT);

    unsigned long prevSrcBlend, prevDestBlend;
    m_pd3dDevice->GetRenderState(D3DRS_SRCBLEND, &prevSrcBlend);
    m_pd3dDevice->GetRenderState(D3DRS_DESTBLEND, &prevDestBlend);
    m_pd3dDevice->SetRenderState(D3DRS_SRCBLEND, D3DBLEND_SRCALPHA);
    m_pd3dDevice->SetRenderState(D3DRS_DESTBLEND, D3DBLEND_INVSRCALPHA);
    m_pd3dDevice->SetRenderState(D3DRS_ALPHATESTENABLE, false);

    m_pd3dDevice->SetTextureStageState(0, D3DTSS_TEXCOORDINDEX, 0);
    m_pd3dDevice->DrawPrimitiveUP(D3DPT_TRIANGLESTRIP, 2, vertex, sizeof(TEXTUREVERTEX));

    m_pd3dDevice->SetRenderState(D3DRS_SRCBLEND, prevSrcBlend);
    m_pd3dDevice->SetRenderState(D3DRS_DESTBLEND, prevDestBlend);
    m_pd3dDevice->SetRenderState(D3DRS_ALPHABLENDENABLE, false);
    m_pd3dDevice->SetTextureStageState(1, D3DTSS_TEXCOORDINDEX, 1);
    m_pd3dDevice->SetTexture(0, nullptr);

    return true;
}
```

> Note: New Code: Saves and restores render states (D3DRS_SRCBLEND, D3DRS_DESTBLEND), ensuring the rendering pipeline is not disrupted.
New Code: Explicitly configures texture stage states and resets them properly, ensuring predictable behavior. <br>
Similar issues can arise from RenderTextureTriangle if WheelOfFortune, or RenderFillZone because alpha render state isn't being reset properly after the call to those functions. (Can cause some graphical glitches elsewhere).

