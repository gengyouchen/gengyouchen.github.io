---
layout: post
title:  "[AtCoder Beginner Contest 139] 賽後解說"
date:   2019-09-01 21:40:00 +0800
categories: algorithm atcoder
permalink: /atcoder-beginner-contest-139/
comments: yes
---
{% include facebook.html %}

先說明我不是 ACMer, 大學也不是念資工系, 這篇文比較適合出社會後才開始學習演算法的小夥伴們.
希望能幫助到更多跟我一樣平庸的人, 在工作之餘多讀書/多刷題提高自己的演算法水平.

這週的 Beginner Contest 前三題基本都是給大家簽到的性質, 沒空的小夥伴可以從第四題開始寫.

| Problem                                           | Topics                                                                 | Difficulty |
| ------------------------------------------------- | ---------------------------------------------------------------------- | ---------- |
| [A - Tenki (天氣)](#a---tenki-天氣)               | `String`                                                               | Easy       |
| [B - Power Socket (插座)](#b---power-socket-插座) | `Math`                                                                 | Easy       |
| [C - Lower (低處)](#c---lower-低處)               | `Array` `Two Pointers`                                                 | Easy       |
| [D - ModSum (餘數之合)](#d---modsum-餘數之合)     | `Math`                                                                 | Easy       |
| [E - League (聯賽)](#e---league-聯賽)             | `Breadth-first Search` `Depth-first Search` `Graph` `Topological Sort` | Medium     |
| [F - Engines (引擎)](#f---engines-引擎)           | `Geometry` `Sliding Window` `Sort` `Two Pointers`                      | Medium     |

這週還蠻歡樂向的, 對照 LeetCode 的難度, 大概是 Easy ~ Medium, 沒有什麼噁心的題目.

好, 我們開始做題吧!

A - Tenki (天氣)
================

題目: [[AtCoder Beginner Contest 139] A - Tenki](https://atcoder.jp/contests/abc139/tasks/abc139_a?lang=en)

第一題是一個簽到題, 主要的功用是讓不會日文的小夥伴學一下日文 "天気" 讀作 "Tenki".

給你三天的天氣預報 `S` 以及三天的實際天氣 `T`, 只是要你數一下有幾天的天氣預報是準的.

```cpp
class Solution {
public:
	/* time: O(1), space: O(1) */
	int tenki(const string& S, const string& T) {
		int ans = 0;
		for (int i = 0; i < 3; ++i) {
			if (S[i] == T[i])
				++ans;
		}
		return ans;
	}
};
```
完整程式碼: [counting-solution.cpp](https://github.com/gengyouchen/atcoder/blob/master/beginner-contest-139/a-tenki/counting-solution.cpp)

B - Power Socket (插座)
=======================

題目: [[AtCoder Beginner Contest 139] B - Power Socket](https://atcoder.jp/contests/abc139/tasks/abc139_b?lang=en)

第二題也是一個簽到題, 主要的功用是讓英文不好的小夥伴學一下延長線叫作 "Power Strip".

你家原本有一個插座, 你想用延長線把它變成至少 `b` 個插座. 每一條延長線可以提供 `a` 個插座, 問你至少需要買幾條延長線?

延長線本身也需要吃掉一個插座, 所以每一條延長線可增加 `a - 1` 個插座, 已知你需要增加 `b - 1` 個插座, 相除就是答案.

```cpp
class Solution {
public:
	/* time: O(1), space: O(1) */
	int powerSocket(int a, int b) {
		const int needs = (b - 1), unit = (a - 1);
		return (needs + (unit - 1)) / unit;
	}
};
```

完整程式碼: [math-solution.cpp](https://github.com/gengyouchen/atcoder/blob/master/beginner-contest-139/b-power-socket/math-solution.cpp)

C - Lower (低處)
================

題目: [[AtCoder Beginner Contest 139] C - Lower](https://atcoder.jp/contests/abc139/tasks/abc139_c?lang=en)

第三題也是一個簽到題, 陣列 `h` 代表一維地圖上每一個點的高度, 你可能降落在任何一個點, 降落後你只能往右走, 而且不能爬山, 問你最遠能走幾步?

簡單的雙指標做法就能在 `O(n)` 時間內得到答案:
* 令降落後的起點為 `L`, 目前的位置為 `R`, 一開始我們降落在地圖最左端, 不斷向右走 (`++R`), 直到卡住為止 (`h[R] < h[R + 1]`).
* 每次卡住時, 我們可以把起點 `L` 直接跳到從 `R + 1` 出發, 不需要嘗試從 `[L + 1, R]` 之間任一點出發了, 因為他們最終也都會卡在 `R`, 而且走的步數更少, 不會得到更好的結果.

```cpp
class Solution {
public:
	/* time: O(n), space: O(1) */
	int lower(const vector<int>& h) {
		const int n = h.size();
		int ans = 0, L = 0;
		for (int R = 0; R < n; ++R) {
			ans = max(ans, R - L);
			if (R < n - 1 && h[R] < h[R + 1])
				L = R + 1;
		}
		return ans;
	}
};
```

完整程式碼: [two-pointers-solution.cpp](https://github.com/gengyouchen/atcoder/blob/master/beginner-contest-139/c-lower/two-pointers-solution.cpp)

D - ModSum (餘數之合)
=====================

題目: [[AtCoder Beginner Contest 139] D - ModSum](https://atcoder.jp/contests/abc139/tasks/abc139_d?lang=en)

第四題是一道簡單的數學題, 給一個正整數 `n`, 請重新排列 `P[n] = {1, 2, 3, ..., n}` 這串數字的順序, 使 `sum = (P[0] % 1) + (P[1] % 2) + (P[3] % 3) + ... + (P[n - 1] % n)` 最大化.

我們觀察到, 上述 `sum` 裡面每一項 `(P[i] % j)` 的值不可能超過 `(j - 1)`, 如果我們要使 `sum` 最大化, 我們希望讓 `sum` 裡面的每一項 `(P[i] % j)` 都恰好等於 `(j - 1)`, 唯一符合這個要求的排列組合就是 `P[n] = {n, 1, 2, 3, ..., n - 1}`.

此時 `sum = 0 + 1 + 2 + ... + (n - 1)` 是一個等差級數, 數學家高斯在九歲的時候就告訴我們了, 答案是 `n * (n - 1) / 2`.

```cpp
using LL = long long;

class Solution {
public:
	/* time: O(1), space: O(1) */
	LL modsum(int n) {
		/*
		 * The optimal permutation is:
		 *    n    %    1    =    0
		 *    1    %    2    =    1
		 *    2    %    3    =    2
		 *    3    %    4    =    3
		 *    .         .         .
		 *    .         .         .
		 *    .         .         .
		 * (n - 3) % (n - 2) = (n - 3)
		 * (n - 2) % (n - 1) = (n - 2)
		 * (n - 1) %    n    = (n - 1)
		 */
		return (LL)n * (n - 1) / 2;
	}
};
```

完整程式碼: [math-solution.cpp](https://github.com/gengyouchen/atcoder/blob/master/beginner-contest-139/d-modsum/math-solution.cpp)

E - League (聯賽)
=================

題目: [[AtCoder Beginner Contest 139] E - League](https://atcoder.jp/contests/abc139/tasks/abc139_e?lang=en)

第五題是考你拓樸排序, 類似的題目在 LeetCode 上面有很多, 例如:
* [LeetCode 207 - Course Schedule](https://leetcode.com/problems/course-schedule/description/)
* [LeetCode 329 - Longest Increasing Path in a Matrix](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/description/)

這題是給你 `n` 位選手, 每一位選手都需要與其他 `n - 1` 位選手比賽, 但每一位選手心中都各自有一個期望比賽的順序:
對於編號為 `i` 的選手來說, 他心目中期望遇到的第一位對手是編號為 `A[i][0]` 的選手, 第二位對手是 `A[i][1]`, 第三位對手是 `A[i][2]`, ..., 最後一位對手是 `A[i][n - 2]`

這題要問你:
* 是否存在一個比賽的順序, 能滿足每一位選手的期望?
* 如果存在, 請問幾天才能比完? (假設場地無限大, 但是每位選手一天最多只能比賽一場)

我們可以把每一場比賽 (`playerX` v.s. `playerY`) 視為有向圖中的每一個節點, 節點與節點之間如果有順序相依性, 就建立有向邊連起來:
* 每一個節點的編號為 `playerX * n + playerY`
* 令 `playerX < playerY`, 總共只有 `n * (n - 1) / 2` 個有效節點, 其餘節點不會影響計算結果 (因為沒有順序相依性)

對所有節點進行拓樸排序, 拓樸排序可以在 `O(|V|+|E|)` 時間內完成, 對這題來說, 就是 `O(n^2)`.

拓樸排序可以使用 BFS (Kahn's algorithm) 或者 DFS (reverse post-order) 實作, 在此以 BFS 為例:

```cpp
class Solution {
public:
	/* time: O(n^2), space: O(n^2) */
	int league(const vector<vector<int>>& A) {
		/* Build the dependency graph of all matches */
		const int n = A.size();
		vector<vector<int>> adjLists(n * n);
		for (int i = 1; i <= n; ++i) {
			int prevMatch = -1;
			for (int j : A[i - 1]) {
				const int playerX = min(i - 1, j - 1), playerY = max(i - 1, j - 1);
				const int currMatch = playerX * n + playerY;
				if (prevMatch != -1)
					adjLists[prevMatch].push_back(currMatch);
				prevMatch = currMatch;
			}
		}

		/* Compute the length of the topological ordering (Kahn's algorithm) */
		vector<int> inDegree(n * n);
		for (const auto& adjList : adjLists) {
			for (int v : adjList)
				++inDegree[v];
		}

		queue<int> q;
		for (int u = 0; u < n * n; ++u) {
			if (!inDegree[u])
				q.push(u);
		}

		int ans = 0, nVisited = 0;
		for (int depth = 1; !q.empty(); ++depth) {
			for (int i = q.size(); i > 0; --i) {
				const int u = q.front();
				++nVisited, q.pop();
				for (int v : adjLists[u]) {
					if (inDegree[v] && !--inDegree[v])
						q.push(v);
				}
			}
			++ans;
		}
		if (nVisited < n * n)
			return -1; /* failed */
		return ans;
	}
};
```

完整程式碼: [BFS-solution.cpp](https://github.com/gengyouchen/atcoder/blob/master/beginner-contest-139/e-league/BFS-solution.cpp)

F - Engines (引擎)
=================

題目: [[AtCoder Beginner Contest 139] F - Engines](https://atcoder.jp/contests/abc139/tasks/abc139_f?lang=en)

第六題給你 `n` 個向量, 請你挑選其中的某些向量, 組合成一個長度最長的向量 `v`.

這一題本質上是一個滑動窗口的問題, 類似的題目在 LeetCode 上面有很多, 例如:
* [LeetCode 3 - Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/description/)
* [LeetCode 76 - Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/description/)

但這一題就難在: 你未必能想到最佳解就包含在滑動窗口裡面, 這需要一點數學幾何的觀察.

假設這個組合出來的最終向量 `v`, 長度為 `len`, 角度為 `degree` 度:
* 對於題目給我們的 `n` 個向量中, 角度在 `degree - 90` ~ `degree + 90` 之間的向量, 一旦投影到 `degree` 這個方向上, 都會給 `len` 帶來正面的貢獻, 是 `v` 的一部分.
* 其餘角度的向量, 都會給 `len` 帶來負面的貢獻, 不會是 `v` 的一部分.

所以我們把長度為 `n` 個陣列按照每個向量的角度排好序, 答案就會是其中某個子區間 `[L, R]` 內的所有向量之合 (這些向量的角度差距不會超過 180 度), 而這就是一滑動窗口的問題.
* 注意: 子區間 `[L, R]` 可以跨過 360 度 (`R >= n`), 只要 `R < L + n` 就是合法的.

滑動窗口只需要 `O(n)` 時間就能求解, 但由於先前排序就花了 `O(n*log(n))` 的時間, 所以這題的時間複雜度還是 `O(n*log(n))`, 而空間複雜度則是 `O(1)` (因為我們用堆積排序, 不需要額外的輔助空間)

```cpp
using LL = long long;
using Arrow = complex<LL>;

class Solution {
public:
	/*
	 * time: O(n*log(n)), space: O(1) auxiliary (i.e. does not count input itself),
	 * assuming sqrt(x) and atan2(x) are in O(1) time
	 */
	double engines(vector<Arrow>& arrows) {
		const int n = arrows.size();
		/*
		 * Assume the final answer is v = Arrow(x, y), where len = abs(v), degree = arg(v)
		 * All the arrows within (degree - 90) ~ (degree + 90) will contribute to this len.
		 * Therefore, if we sort the array of arrows by degree, the answer is one of its subarrays.
		 */
		auto compareAngle = [](const Arrow& a, const Arrow& b) {
			return atan2(imag(a), real(a)) < atan2(imag(b), real(b));
		};
		make_heap(arrows.begin(), arrows.end(), compareAngle);
		sort_heap(arrows.begin(), arrows.end(), compareAngle);

		Arrow window(0, 0);
		LL ans = 0;
		int R = 0;
		for (int L = 0; L < n; ++L) {
			if (R < L)
				R = L, window = Arrow(0, 0);
			while (R - L < n && norm(window + arrows[R % n]) >= norm(window))
				window += arrows[R % n], ++R;
			ans = max(ans, norm(window)), window -= arrows[L % n];
		}
		return sqrt(ans);
	}
};
```

完整程式碼: [sliding-window-solution.cpp](https://github.com/gengyouchen/atcoder/blob/master/beginner-contest-139/f-engines/sliding-window-solution.cpp)

{% if page.comments %}
<div class="fb-comments" data-href="{{ site.url }}{{ page.url }}" data-width="100%" data-numposts="5"></div>
{% endif %}
