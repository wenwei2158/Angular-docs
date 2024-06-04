# 建立第三層留言區方法與動態插入元件遇到的困難
首先，原本設計只有討論區以及討論區底下第二層回覆區。現在==新增三層留言區的方法是將第二層與第三層留言區打包成一個元件==，原因在於如果在原元件新增第三層留言會導致維護以及修改不易因此獨立出元件。

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
**由於要將訊息框變成[動態插入元件](https://fullstackladder.dev/blog/2022/01/16/dynamic-create-component-without-template/)，則需要以下步驟才能達成**
>[!Note]
提醒:Angular 13後不再提供ComponentFactoryResolver，ViewContainerRef 本身的 createComponent() 可以直接指定元件

*這邊有三個步驟*

### 2-1. 決定要拔除的對象
    
1. 將原先回覆框(reply-field.component)生成另一個元件
2. 把template, component分離
3. **==除錯==**:在分離元件時會碰到資料流以及方法內部傳遞值的問題，可以將必要及非必要的物件以及方法釐清再做更動以免搞混以及<font color="#f00">ERROR</font>
4. 由於拔除對象內需要引入參數值，因此新增==interface==(介面)一方面提供引用，另一方面因為會需要利用服務元件將需要用到的數值傳入，[Injection Token](https://xie.infoq.cn/article/84e445cb9276c6f8a7ec13c61)派上用場。


### 2-2.動態插入元件會需要標記位置，因此建立service儲存資料
 
1. 首先建立service，服務元件是資料注入的地方，因此也需要匯入剛才所建立的```injection token```以及```interface```。
2. 由於回覆框可以在不同元件內使用，因此需要判斷標記位置的做法。我們利用```ViewContainerRef```標記位置，給定原先、後來的位置，在要產生對話框時將當前位置remove()，之後將後來的位置覆蓋當前的位置，產生回覆框元件，也可以做到在不同位點擊回覆時可以只在單一位置生成。

```
export class ReplyFieldInsertorService {
  currentPortal?: ViewContainerRef
  constructor() {return;}
  insertReplyField(
    portalFromMarker: ViewContainerRef,
    replyContext: ReplyContext
  ) {
    // if (portalFromMarker === this.currentPortal) {
    //   return;
    // }
    this.currentPortal?.remove();
    this.currentPortal = portalFromMarker;
    return this.currentPortal.createComponent(ReplyFieldComponent, {
      injector: Injector.create([
        {
          provide: REPLY_CONTEXT,
          useValue: replyContext,
        },]),});}} 
```
![螢幕擷取畫面 2024-06-04 141536](https://hackmd.io/_uploads/r1K3KS3ER.png)
<center>(示意程式碼)</center>


### 2-3.最後一項也是最重要**directive**
:::info
(目的在於它可以利用```directive```的特性與```structural directive```放置在一起，依照你所綁定的方法將他動態生成在畫面上) ps.這邊的方法是利用HtmlButtonElement綁定對應按鈕
:::
1. 當按鈕發生動作時```fromevent```監聽事件產生，接著將回覆框動態插入到```viewContainerRef```，並傳遞回覆框內容的值。當成功插入組件，會觸發另一個```observable```，讓外部讀取。

```
export class ReplyMarkerDirective implements OnInit {
  @Input() trigger?: HTMLButtonElement; //viewchild
  @Input() replyContext?: ReplyContext;
  @Output() refreshReplies: EventEmitter<void> = new EventEmitter<void>()
  constructor(
    private viewContainerRef: ViewContainerRef,
    private insertor: ReplyFieldInsertorService
  ) {
    return;}
  ngOnInit(): void {
    if (!this.trigger) return;
    const _context = this.replyContext;
    if (!_context) return;
    //就是根據你要監聽的事件，發送相關的事件資料給你
    fromEvent(this.trigger, 'click').subscribe({
      next: () => {
        const componentRef = this.insertor.insertReplyField(this.viewContainerRef, _context);
        componentRef?.instance.refreshReply$.subscribe({
          next: () => {
            this.refreshReplies.emit()}})}})}}
```
![螢幕擷取畫面 2024-06-04 144615](https://hackmd.io/_uploads/B1wpYBnEA.png)
<center>(示意程式碼)</center>

3.第三步:將元件拆解
-
![畫面顯示](https://hackmd.io/_uploads/HymHWfPNC.png)
<center>(示意畫面顯示)</center>
<br>
    
紅色方框表示討論區第一層，黃色方框表示第二與第三層留言，我們要做的第一件事就是將他們拆開成兩個元件。


