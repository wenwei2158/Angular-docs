# 建立第三層留言區方法與動態插入元件遇到的困難
首先，原本設計只有討論區以及討論區底下第二層回覆區。現在新增三層留言區的方法是將第二層與第三層留言區打包成一個元件，原因在於如果在原元件新增第三層留言會導致維護以及修改不易因此獨立出元件。

再來是將回覆框動態插入在可能產生回覆留言的第一二層內。這邊會需要使用到比較困難的方法，因此特別提出來做說明。

1.第一步:加入動態產生元件
-
![程式碼](https://hackmd.io/_uploads/r1gVpmfP40.png)
<center>(示意程式碼)</center>
<br>

紅色方框表示thread_head-comment討論區第一層，黃色方框thread_reply-list表示第二層留言區，此時尚未加入第三層。
cgustMarker表示動態生成訊息框元件。

2.第二步:元件拆解分成幾個步驟:
-
**由於要將訊息框變成[動態插入元件](https://fullstackladder.dev/blog/2022/01/16/dynamic-create-component-without-template/)，則需要以下步驟才能達成
> 提醒:Angular 13後不再提供ComponentFactoryResolver，ViewContainerRef 本身的 createComponent() 可以直接指定元件**

1. 決定要拔除的對象
2. 動態插入元件會需要標記位置，因此建立directive以及service儲存資料
3. 檢查是否有與原元件相關的方法、參數等資料，以利拆解




(更新中)
3.第三步:將元件拆解
-
![畫面顯示](https://hackmd.io/_uploads/HymHWfPNC.png)
<center>(示意畫面顯示)</center>
<br>
    
紅色方框表示討論區第一層，黃色方框表示第二與第三層留言，我們要做的第一件事就是將他們拆開成兩個元件。

