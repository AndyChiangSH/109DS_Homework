# 資料結構作業04_解題報告

###### tags: `資料結構` `C`

## Q1 Symbolic Equations
這題比較像在考圖的問題，用到的是資料結構中的==Adjacency Lists==。

### 結構
我們會增設兩種解點：分別是==標頭節點==和==清單節點==

:::spoiler 詳細節點結構
```
typedef struct headNode *hptr;
typedef struct listNode *lptr;

typedef struct headNode		//變數的標頭
{
	char var[2];
	lptr next;
	lptr top;
	char func[2];
}headNode;
typedef struct listNode		//方程式內的變數 
{
	short int visited;
	hptr rel;	//會指向其他變數的標頭 
	lptr next;
}listNode;
```
:::

### 演算法
![](https://i.imgur.com/NwwkVIR.jpg)
簡單講就是利用遞迴，跑過整個關係圖，從一個變數的標頭點開始，將它底下所有的清單節點跑過一遍，每個清單節點內的指標都可以再連到其他的標頭點，至少有一個變數的標頭點沒有清單節點，遞迴最後都會停在這些沒有清單節點的標頭點上。

* **第一次檢查有沒有迴圈(待更正)**
我這裡的演算法比較爛一點，只要同一條路徑經過兩次，就當作有迴圈，很明顯地是有問題的
* **第二次印出方程式展開**
如果第一次的結果說沒有迴圈，才會進入這個Part，一樣是遞迴，如果是沒有清單節點的標頭點，就印出自己。否則，先印出方程式名稱和`(`，跑過所有清單節點，最後印出 `)`。

## Q2 A Sticky Safe Lock
把這題想成HW01的1-1就很簡單了，只不過從二維變成四維，原理不變。
### 結構
dial是一個代表轉輪的四維陣列，其中數字的涵義：
* -1代表會爆炸的數字(視為牆壁)
* 0代表可以走的空位
* 1代表終點

bmark是用來記錄廣度搜尋下，與起點的距離，起點是0，也是四維陣列。

:::spoiler 詳細結構
```
short int dial[MAX_DIAL][MAX_DIAL][MAX_DIAL][MAX_DIAL];	//轉輪，-1代表爆炸，1代表終點，其餘為0 
short int bmark[MAX_DIAL][MAX_DIAL][MAX_DIAL][MAX_DIAL];	//紀錄廣度的距離 
```
:::

---
為了方便，我將八種方向給結構化，命名為offsets。

```
[0] +1 +1  0  0    //d1向上撥
[1] -1 -1  0  0    //d1向下撥
[2] +1 +1 +1  0    //d2向上撥
[3] -1 -1 -1  0    //d2向下撥
[4]  0 +1 +1 +1    //d3向上撥
[5]  0 -1 -1 -1    //d3向下撥
[6]  0  0 +1 +1    //d4向上撥
[7]  0  0 -1 -1    //d4向下撥
```

:::spoiler 詳細節點結構
```
typedef struct		//結構: 八個方向
{
	short int d1;
	short int d2;
	short int d3;
	short int d4;
}offsets;
offsets move[8];
```
:::

---

另外因為等一下要用廣度搜尋，所以建立好一個Queue。

:::spoiler 詳細節點結構
```
typedef struct Qnode* qptr;
struct Qnode	//Queue節點
{
	int len;
	short int d1;
	short int d2;
	short int d3;
	short int d4;
	qptr next;
}qnode;
qptr front=NULL,rear=NULL;
```
:::

---

### 演算法
如同1-1的找出最短路徑，我們使用==廣度搜尋法==

從起點開始，每次先從Queue中取出最前面的元素，取出的元素裡會提供他的座標，利用他的座標往八個方向去嘗試，會有幾種情況：

* -1 會爆炸的數字(視為牆壁)，不可移動
* 0 可以移動，將新的點加入Queue的尾端
* 1 終點，紀錄最短路徑並結束迴圈

以前迷宮作法是在周圍圍上圍牆，但因為這題的0會變成9，9會變成0，所以不能用這招。替代方法是：==(下一個位置+10)%10==。

:::spoiler 詳細程式碼
```
nextd1=(current->d1+move[dir].d1+10)%10;
nextd2=(current->d2+move[dir].d2+10)%10;
nextd3=(current->d3+move[dir].d3+10)%10;
nextd4=(current->d4+move[dir].d4+10)%10;
```
:::

---

印出轉輪過程就如同1-1的印出路徑，只不過因為是從終點開始回推，最後要反向印而已。

廣度搜尋的部分不再多做介紹： 可以回去看 [資料結構作業01_解題報告](/@AndyChiang/rkp_n-Euv)

## Q3 Haunted Mansion
這題稍難，想成==深度搜尋+樹==會比較簡單。

### 結構
豪宅地圖說穿了就是兩個n\*n的陣列，一個(map)用來實際更改，一個(origin)用來保存圖的原貌。

:::spoiler 詳細結構
```
char map[N][N];	//豪宅地圖 
char origin[N][N];	//維持地圖原狀，程式執行時不會影響他
```
:::

---

另外一個陣列(visible)，用來存四邊窗戶所看到的怪物數量。

:::spoiler 詳細結構
```
int visible[4][N];	//四邊窗戶看到的怪物數量 
for(i=0;i<4;i++)
{
	for(j=0;j<N;j++)
	{
		fscanf(fin,"%d",&visible[i][j]);
	}
}
```
:::

---

### 演算法
關鍵在於兩個函式：==lookMap== 和 ==lookInfo==

* *lookMap*：印出整個地圖(map)
* *lookInfo*：從每一個窗戶開始跑，計算視線路徑中**看的到的**怪物數量，並和visible做比較。
    * 回傳0 -> 有一條視線有錯
    * 回傳1 -> 無論是否已經填滿，所有已經連成一條的視線都是對的

深度搜尋以遞迴做實現，函式名dfs，會回傳0和1：

* 回傳0 -> 代表這條路以下都是錯的
* 回傳1 -> 代表已經有解答了

![](https://i.imgur.com/cACve9J.jpg)
想像成這是一棵樹，每個節點底下最多可以有三個子節點，遞迴最初會傳入三種怪物的數量，如果吸血鬼的數量大於0，就把該格設為吸血鬼，並把吸血鬼的數量減一，傳給下一個遞迴，其他怪物也是依此類推，**特別注意，三種怪物的程序是平行處理的，所以有可能一個遞迴呼叫三個遞迴**。

如果三種怪物的數量都為0時，代表空格全部都填滿了，於是呼叫*lookInfo*檢查是否正確，正確回傳1，錯誤回傳0。


做到這裡已經可以應付第一二筆測資了，但三四筆測資會因為深度太深而跑很久，因此優化方式如下：

![](https://i.imgur.com/sNyVD8T.jpg)

一進入遞迴，先檢查地圖上有沒有錯誤的視線，如果有一條以上是錯誤的，這個節點的子樹也會全部都是錯誤的，因此遞迴在此不要再往下傳了，直接回傳0，上一層遞迴如果接到0，會將地圖回復為空白(`.`)，如此一來，遞迴次數便會減少許多。

## Q4 Baba is You
🛠施工中... 

> 看完覺得有幫助的話，就幫我點個右上角的star吧! 你的鼓勵是我繼續創作的原動力~

by @AndyChiang 

> [time=Wed, Jul 14, 2021 7:36 PM]