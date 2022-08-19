# Disable NPC Dialogue Option  
_The chat bubbles can get annoying for the player and this has been quite well recieved._

---
## Adding parts to the source
---
   - **HWOption.h**  
	_Somewhere in class COption but under public._
     ```CPP
		bool ignoreNpcDialogue = false;
     ```   	
   - **HWOption.cpp**  
	_Add to the Load function under another option._
     ```CPP
		else if (scan.Token == _T("IgnoreNpcDialogue"))
			ignoreNpcDialogue = scan.GetNumber() != 0;
     ```
     _Add to the Save function under another option._
     ```CPP
		_ftprintf(fp, _T("IgnoreNpcDialogue %d\n"), static_cast<int>(ignoreNpcDialogue));
     ```   	
   - **DPClient.cpp (Neuz)**  
	in: CDPClient::OnChat
     ```CPP
		if (strlen(szBuf) && (szBuf[0] == '/' || szBuf[0] == '!'))
		{
			if (pMover->IsNPC() && g_Option.ignoreNpcDialogue)
				return;
			g_DialogMsg.AddMessage(pMover, szBuf);
		}
		else
		{
			if (pMover->IsNPC() && g_Option.ignoreNpcDialogue)
				return;
			sprintf_s(szChat, "%s : %s", pMover->GetName(true), szBuf);
			g_WndMng.PutString(szChat, pMover, 0xffffffff, CHATSTY_GENERAL);
		}
     ```   	


## Thank you
---
_FlyFF is owned and operated and developed by Galanet/Aeonsoft. This is only an implementation made by myself, Kia#1411._

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)

