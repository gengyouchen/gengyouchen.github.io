---
layout: post
title:  "[Google Kick Start 2019 Round E] 賽後解說"
date:   2019-08-25 16:00:00 +0800
categories: algorithm kickstart
permalink: /google-kick-start-2019-round-e/
comments: yes
---
{% include facebook.html %}

先聲明我不是 ACMer, 大學也不是念資工系, 我的觀點比較適合出社會後才開始學習演算法的小夥伴. 希望我的文章能幫助到許多跟我一樣平庸的人, 在工作之餘多多刷題提高自己的演算法水平.

這個月的 Kick Start 比較中規中矩, 除了第一題跟 LeetCode 原題有 87 分像, 剩下兩題也都只需要簡單的分析, 沒有要刁難我們的意思.

| Problem                                                         | Topics                                                            | Difficulty |
| --------------------------------------------------------------- | ----------------------------------------------------------------- | ---------- |
| [Cherries Mesh (櫻桃網)](#cherries-mesh-櫻桃網)                 | `Breadth-first Search` `Depth-first Search` `Graph` `Union Find`  | Medium     |
| [Code-Eat Switcher (碼食轉換家)](#code-eat-switcher-碼食轉換家) | `Binary Search` `Greedy`  `Sort`                                  | Medium     |
| [Street Checkers (街頭跳棋)](#street-checkers-街頭跳棋)         | `Math`                                                            | Medium     |

對照 LeetCode 的難度, 大概也是 Medium ~ Hard, 但這個月比起以往的 Kick Start 偏簡單, 請允許我都標 Medium 就好, 不要自己嚇自己.

好, 讓我們開始讀題吧!

Cherries Mesh (櫻桃網)
======================

題目: [[Google Kick Start 2019 Round E] Cherries Mesh](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050edb/0000000000170721)

第一題是考你無向圖的連通元件數目, 類似的題目在 LeetCode 上面有很多, 例如:
* [LeetCode 200 - Number of Islands](https://leetcode.com/problems/number-of-islands/description/)
* [LeetCode 323 - Number of Connected Components in an Undirected Graph](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/description/)

這題是給你 `n` 顆櫻桃組成的網, 任兩個櫻桃之間, 有些用了黑色糖線相連 (題目已給你 `m` 條黑色糖線), 剩下的則是用了紅色糖線相連, 每條黑色糖線含有 `1` 單位的糖, 而每條紅色糖線含有 `2` 單位的糖.

題目要你盡可能移除多餘的糖線, 只要能保持任兩個櫻桃之間都有直接或間接的相連即可, 問你這張網最少會剩下多少單位的糖.

要將 `n` 顆櫻桃直接或者間接的相連, 最少只需要 `n - 1` 條糖線, 要計算糖的數目, 我們只需要知道其中哪些糖線是紅色的.
* 如果我們忽略紅色糖線, 只看黑色糖線, 那麼任兩個櫻桃之間, 只要可以用黑色糖線直接或者間接相連, 就屬於同一個連通元件. 在同一個連通元件內, 不需要保留任何紅色糖線.
* 假設圖上有 `k` 個不同的連通元件, 我們只需要 `k - 1` 條紅色糖線即可. 所以整張櫻桃網的糖總數, 就是 `(n - 1) + (k - 1)` 個單位.
* 計算圖上的連通元件數目, 只需要 `O(|V| + |E|)` 的時間, 對這題來說, 就是 `O(n + m)`.

尋找圖上的所有連通元件, 可以選用 DFS (stack), BFS (queue), 或者 Union Find (Disjoint Sets) 來實作, 在此以 DFS 為例:
```cpp
class Solution {
public:
	/* time: O(n + m), space: O(n + m) */
	int cherriesMesh(int n, const vector<pair<int, int>>& blacks) {
		vector<vector<int>> adjLists(n);
		for (const auto& black : blacks)
			adjLists[black.first - 1].push_back(black.second - 1);

		vector<int> visited(n);
		auto visit = [&](int src) {
			if (visited[src])
				return false;
			stack<int> s;
			visited[src] = true, s.push(src);
			while (!s.empty()) {
				const int u = s.top();
				s.pop();
				for (int v : adjLists[u]) {
					if (!visited[v])
						visited[v] = true, s.push(v);
				}
			}
			return true;
		};

		int count = 0;
		for (int u = 0; u < n; ++u) {
			if (visit(u))
				++count;
		}
		return (n - 1) + (count - 1);
	}
};
```

完整程式碼: [DFS-solution.cpp](https://github.com/gengyouchen/kickstart/blob/master/2019-round-e/cherries-mesh/DFS-solution.cpp)

Code-Eat Switcher (碼食轉換家)
==============================

題目: [[Google Kick Start 2019 Round E] Code-Eat Switcher](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050edb/00000000001707b8)

第二題是一個日常時間分配的問題, 憑生活經驗就能很容易想到正確的貪心策略, 但實作上, 需要二分搜索才能通過大測資.

題目把一天分成 `s` 個時段, 每個時段你都有不同的工作效率/進食效率 (這很符合人類的現實狀況):
* 已知第 `j` 個時段 (`slots[j]`) 你全部拿來寫程式可達成的工作量 (`slots[j].coding`), 以及你全部拿來吃飯可達成的進食量 (`slots[j].eating`).
* 對於每一個時段, 你都可以自由選擇要拿多少比例的時間來寫程式 (`p%`), 拿多少比例的時間來吃飯 (`1 - p%`).

題目要問你: 有彼此獨立的 `d` 天, 請你查詢每一天你是否能達成目標?
* 已知第 `i` 天 (`days[i]`) 你的總工作量目標 (`days[i].coding`), 以及你的總進食量目標 (`days[i].eating`).
* 每一天的查詢是彼此獨立的, 請不用考慮第 `i` 天做不完, 會不會影響到第 `i + 1` 天的問題.

從題目的敘述, 我們很容易想到一個貪心策略:
* 對於時段 `j`, 少吃一單位食物, 能多做的工作量是 `slots[j].coding / slots[j].eating`, 我們可以把這個當作時段 `j` 的工作性價比 (C/P ratio).
* 我們可以預先將所有時段 `slots` 按照工作性價比由高到低排序 (需要 `O(s*log(s))` 的時間), 然後優先把工作性價比高的時間分配給工作, 直到總工作量目標達成為止.
* 如果總工作量目標達成後, 剩下的時間不足以完成總進食量目標, 那麼就保證這天無法完成目標: 因為我們已經選工作性價比最高的時間了, 其他的分配方法在滿足總工作量目標後, 剩下的時間肯定更少.

對於已經排序完的所有時段 `slots`, 要實作這個貪心策略, 我們需要很有效率的找出最小的 `k` 值使得 `sum(slots[0].coding ~ slots[k].coding) >= days[i].coding`:
* 我們可以預先計算 `slots` 的前綴和 `prefix`, 這需要 `O(s)` 的時間, 但是這是一次性的時間成本, 之後 `d` 天的查詢都可以重複利用這個 `prefix`.
* 要找出每一天對應的最小 `k` 值, 只需要二分搜索 `prefix[k].coding >= days[i].coding` 即可, 每天只需要 `O(log(s))` 的時間, 總共 `d` 天就是 `O(d*log(s))` 的時間.

所以總時間複雜度是 `O((s + d)*log(s))`, 程式碼實作如下:

```cpp
using LL = long long;

struct Activity {
	LL coding, eating;
	Activity() : coding(0), eating(0) {}
	Activity(LL coding, LL eating) : coding(coding), eating(eating) {}
};

class Solution {
public:
	/*
	 * time: O((s + d) * log(s)), space: O(s) auxiliary (i.e. does not count output itself),
	 * where s = # of slots, d = # of days
	 */
	vector<bool> codeEatSwitcher(vector<Activity>& slots, const vector<Activity>& days) {
		const int d = days.size(), s = slots.size();

		/* order by their C/P ratios (i.e. coding / eating): high to low */
		auto byCPRatio = [](const auto& a, const auto& b) {
			return a.coding * b.eating > b.coding * a.eating;
		};
		sort(slots.begin(), slots.end(), byCPRatio);

		vector<Activity> prefix(s);
		auto sum = [](const auto& a, const auto& b) {
			return Activity(a.coding + b.coding, a.eating + b.eating);
		};
		partial_sum(slots.begin(), slots.end(), prefix.begin(), sum);

		vector<bool> ans(d);
		for (int i = 0; i < d; ++i) {
			const auto& goal = days[i];

			auto byCoding = [](const auto& a, const auto& b) {
				return a.coding < b.coding;
			};
			const auto it = lower_bound(prefix.begin(), prefix.end(), goal, byCoding);
			if (it == prefix.end())
				continue; /* failed: even if we code all the day, we cannot meet our goal */

			const int k = distance(prefix.begin(), it);
			const int lastCoding = goal.coding - (k > 0 ? prefix[k - 1].coding : 0);
			const double firstEating = slots[k].eating - (double)lastCoding * slots[k].eating / slots[k].coding;

			const double totalEating = firstEating + (prefix.back().eating - prefix[k].eating);
			if (totalEating >= goal.eating)
				ans[i] = true;
		}
		return ans;
	}
};
```

完整程式碼: [binary-search-solution.cpp](https://github.com/gengyouchen/kickstart/blob/master/2019-round-e/code-eat-switcher/binary-search-solution.cpp)

Street Checkers (街頭跳棋)
==========================

題目: [[Google Kick Start 2019 Round E] Street Checkers](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050edb/00000000001707b9)

第三題本質上是考你質數判定, 這種題目在 LeetCode 上也有一些:
* [LeetCode 204 - Count Primes](https://leetcode.com/problems/count-primes/description/)
* [LeetCode 1175 - Prime Arrangements](https://leetcode.com/problems/prime-arrangements/description/)

只是題目經過包裝, 需要一點分析才能意識到跟質數有關, 然後套用 [Sieve of Eratosthenes](https://cp-algorithms.com/algebra/sieve-of-eratosthenes.html) 求解.





```cpp
class StreetCheckers {
private:
	vector<int> primes;
	int jumpToStart(int curr, int start, int step) {
		if (curr < start) {
			const int remain = (start - curr) % step;
			curr = start + (remain ? (step - remain) : 0);
		}
		return curr;
	}
public:
	/* time: O(sqrt(M) * log(log(M))), space: O(sqrt(M)), where M = max(R for all test cases) */
	StreetCheckers(int M) {
		const int n = sqrt(M);
		/* Perform the sieve of Eratosthenes in O(n*log(log(n))) time */
		vector<bool> sieve(n + 1, true);
		for (int i = 0; i * i <= n; ++i) {
			if (i < 2) {
				sieve[i] = false;
			} else if (sieve[i]) {
				for (int j = i * i; j <= n; j += i)
					sieve[j] = false;
			}
		}

		for (int i = 0; i <= n; ++i) {
			if (sieve[i])
				primes.push_back(i);
		}
	}

	/* time: O((R - L) * log(log(R))), space: O(R - L) */
	int count(int L, int R) {
		const int R4 = R / 4, L4 = L / 4;
		vector<bool> sieve(R - L + 1, true), sieve4(R4 - L4 + 1, true);
		for (int i : primes) {
			for (int j = jumpToStart(i * i, L, i); j <= R; j += i)
				sieve[j - L] = false;
			for (int j = jumpToStart(i * i, L4, i); j <= R4; j += i)
				sieve4[j - L4] = false;
		}

		/*
		 * Let i = 2^x * 3^y * 5^z * 7^w * ...
		 * Alice will get: (y+1) * (z+1) * (w+1) * ...
		 * Blob will get: x * (y+1) * (z+1) * (w+1) * ...
		 *
		 * Therefore, game i is interesting if-and-only-if:
		 *    (a) x is 0 and (3^y * 5^z * 7^w * ...) is prime or 1
		 * OR (b) x is 1
		 * OR (c) x is 2 and (3^y * 5^z * 7^w * ...) is prime or 1
		 * OR (d) x is 3 and (3^y * 5^z * 7^w * ...) is 1
		 */
		int ans = 0;
		for (int i = L; i <= R; ++i) {
			if (i % 2) {
				if (sieve[i - L])
					++ans;
			} else if (i % 4) {
				++ans;
			} else if (i % 8) {
				if (sieve4[i / 4 - L4])
					++ans;
			} else {
				if (i / 8 == 1)
					++ans;
			}
		}
		return ans;
	}
};
```

完整程式碼: [math-solution.cpp](https://github.com/gengyouchen/kickstart/blob/master/2019-round-e/street-checkers/math-solution.cpp)

{% if page.comments %}
<div class="fb-comments" data-href="{{ site.url }}{{ page.url }}" data-width="100%" data-numposts="5"></div>
{% endif %}
