# 208. 实现 Trie (前缀树)【中等】

## 题目描述

**Trie**（发音类似 "try"）或者说 **前缀树** 是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。这一数据结构有相当多的应用情景，例如自动补完和拼写检查。

请你实现 Trie 类：

- `Trie()` 初始化前缀树对象。
- `void insert(String word)` 向前缀树中插入字符串 `word`。
- `boolean search(String word)` 如果字符串 `word` 在前缀树中，返回 `true`（即，在检索之前已经插入）；否则，返回 `false`。
- `boolean startsWith(String prefix)` 如果之前已经插入的字符串 `word` 的前缀之一为 `prefix`，返回 `true`；否则，返回 `false`。

---

## 输入输出示例

### 示例
```
输入：
["Trie", "insert", "search", "search", "startsWith", "insert", "search"]
[[], ["apple"], ["apple"], ["app"], ["app"], ["app"], ["app"]]

输出：
[null, null, true, false, true, null, true]

解释：
Trie trie = new Trie();
trie.insert("apple");
trie.search("apple");   // 返回 true
trie.search("app");     // 返回 false
trie.startsWith("app"); // 返回 true
trie.insert("app");
trie.search("app");     // 返回 true
```

### 提示
- `1 <= word.length, prefix.length <= 2000`
- `word` 和 `prefix` 仅由小写英文字母组成
- `insert`、`search` 和 `startsWith` 调用次数 **总计** 不超过 `3 * 10^4` 次

---

## 解题思路（面试推荐 - 最优解）

### 核心思想

**什么是Trie（前缀树）？**
```
Trie是一种多叉树结构

特点：
1. 根节点不包含字符
2. 每个节点最多有26个子节点（a-z）
3. 从根到某节点的路径代表一个字符串
4. 每个节点标记是否是单词结尾

例如：插入 "apple", "app", "application"

         root
          |
          a
          |
          p
          |
          p (isEnd=true, "app")
          |
          l
          |
          e (isEnd=true, "apple")
    /           \
   (继续其他分支)

优势：
- 字符串检索：O(m) - m是字符串长度
- 前缀匹配高效
- 空间共享前缀
```

**与哈希表的区别：**
```
哈希表：
- search("apple"): O(1)
- startsWith("app"): O(n) 需要遍历所有键

Trie：
- search("apple"): O(m)
- startsWith("app"): O(m) 只需遍历前缀路径
- 特别适合前缀查询
```

---

### 数据结构设计

**节点结构：**
```typescript
class TrieNode {
    children: Map<string, TrieNode>;  // 子节点映射
    isEnd: boolean;                   // 是否是单词结尾

    constructor() {
        this.children = new Map();
        this.isEnd = false;
    }
}
```

**为什么用Map？**
```
方案1：数组 children[26]
- 优点：访问O(1)
- 缺点：空间浪费（很多为null）

方案2：Map<char, TrieNode>
- 优点：节省空间，只存在的字符
- 缺点：访问O(1)但常数较大

推荐：字符集小用数组，字符集大用Map
```

---

### 操作实现

#### 1. Insert（插入）

**算法流程：**
1. 从根节点开始
2. 对于word中的每个字符：
   - 如果子节点不存在，创建新节点
   - 移动到子节点
3. 标记最后节点为单词结尾

**时间复杂度：** O(m) - m是单词长度
**空间复杂度：** O(m) - 最坏情况需要创建m个新节点

---

#### 2. Search（搜索）

**算法流程：**
1. 从根节点开始
2. 对于word中的每个字符：
   - 如果子节点不存在，返回false
   - 移动到子节点
3. 检查最后节点的isEnd标记

**时间复杂度：** O(m)
**空间复杂度：** O(1)

---

#### 3. StartsWith（前缀匹配）

**算法流程：**
1. 与search类似
2. 区别：不需要检查isEnd标记
3. 只要路径存在即可

**时间复杂度：** O(m)
**空间复杂度：** O(1)

---

## 代码实现（TypeScript）

### 实现一：使用Map

```typescript
class TrieNode {
    children: Map<string, TrieNode>;
    isEnd: boolean;

    constructor() {
        this.children = new Map();
        this.isEnd = false;
    }
}

class Trie {
    private root: TrieNode;

    constructor() {
        this.root = new TrieNode();
    }

    /**
     * 插入单词
     */
    insert(word: string): void {
        let node = this.root;

        // 遍历单词的每个字符
        for (const char of word) {
            // 如果子节点不存在，创建新节点
            if (!node.children.has(char)) {
                node.children.set(char, new TrieNode());
            }
            // 移动到子节点
            node = node.children.get(char)!;
        }

        // 标记单词结尾
        node.isEnd = true;
    }

    /**
     * 搜索单词
     */
    search(word: string): boolean {
        const node = this.searchPrefix(word);
        // 找到节点且是单词结尾
        return node !== null && node.isEnd;
    }

    /**
     * 前缀匹配
     */
    startsWith(prefix: string): boolean {
        const node = this.searchPrefix(prefix);
        // 只要找到路径即可
        return node !== null;
    }

    /**
     * 辅助函数：搜索前缀路径
     * @returns 前缀最后一个字符对应的节点，不存在返回null
     */
    private searchPrefix(prefix: string): TrieNode | null {
        let node = this.root;

        for (const char of prefix) {
            if (!node.children.has(char)) {
                return null;  // 路径不存在
            }
            node = node.children.get(char)!;
        }

        return node;
    }
}
```

---

### 实现二：使用数组（仅小写字母）

```typescript
class TrieNode {
    children: (TrieNode | null)[];  // 26个字母
    isEnd: boolean;

    constructor() {
        this.children = Array(26).fill(null);
        this.isEnd = false;
    }
}

class Trie {
    private root: TrieNode;

    constructor() {
        this.root = new TrieNode();
    }

    /**
     * 字符转索引：'a'→0, 'b'→1, ...
     */
    private charToIndex(char: string): number {
        return char.charCodeAt(0) - 'a'.charCodeAt(0);
    }

    insert(word: string): void {
        let node = this.root;

        for (const char of word) {
            const index = this.charToIndex(char);

            if (node.children[index] === null) {
                node.children[index] = new TrieNode();
            }

            node = node.children[index]!;
        }

        node.isEnd = true;
    }

    search(word: string): boolean {
        const node = this.searchPrefix(word);
        return node !== null && node.isEnd;
    }

    startsWith(prefix: string): boolean {
        return this.searchPrefix(prefix) !== null;
    }

    private searchPrefix(prefix: string): TrieNode | null {
        let node = this.root;

        for (const char of prefix) {
            const index = this.charToIndex(char);

            if (node.children[index] === null) {
                return null;
            }

            node = node.children[index]!;
        }

        return node;
    }
}
```

---

## 代码原理详解

### Insert过程

```
插入 "apple"：

初始：
root

插入'a'：
root → a

插入'p'：
root → a → p

插入'p'：
root → a → p → p

插入'l'：
root → a → p → p → l

插入'e'：
root → a → p → p → l → e (isEnd=true)
```

### Search和StartsWith的区别

```
树结构（插入"apple", "app"）：
root → a → p → p (isEnd=true) → l → e (isEnd=true)

search("app")：
1. 找到节点'p'(第二个p)
2. 检查isEnd = true
3. 返回 true ✓

search("appl")：
1. 找到节点'l'
2. 检查isEnd = false
3. 返回 false ✗

startsWith("appl")：
1. 找到节点'l'
2. 不检查isEnd
3. 返回 true ✓
```

---

## 运行示例

### 示例：完整操作序列

```
操作序列：
trie.insert("apple");
trie.insert("app");
trie.search("apple");
trie.search("app");
trie.search("appl");
trie.startsWith("app");

执行过程：

1. insert("apple")：
   root → a → p → p → l → e*
   （*表示isEnd=true）

2. insert("app")：
   root → a → p → p*
   （复用前缀a→p→p）

当前树结构：
       root
        |
        a
        |
        p
        |
        p* (isEnd=true, "app")
        |
        l
        |
        e* (isEnd=true, "apple")

3. search("apple")：
   路径：root→a→p→p→l→e
   e.isEnd = true
   返回：true ✓

4. search("app")：
   路径：root→a→p→p
   p.isEnd = true
   返回：true ✓

5. search("appl")：
   路径：root→a→p→p→l
   l.isEnd = false
   返回：false ✗

6. startsWith("app")：
   路径：root→a→p→p
   节点存在
   返回：true ✓
```

---

## 常见问题

**Q1: 为什么要有isEnd标记？**
A: 区分前缀和完整单词：
```
插入："app", "apple"

没有isEnd：
无法区分"app"是完整单词还是前缀

有isEnd：
- 节点"app"：isEnd=true（完整单词）
- 节点"appl"：isEnd=false（仅是前缀）
```

**Q2: 如何删除单词？**
A: 需要额外实现delete方法：
```typescript
delete(word: string): boolean {
    return this.deleteHelper(this.root, word, 0);
}

private deleteHelper(node: TrieNode, word: string, index: number): boolean {
    if (index === word.length) {
        if (!node.isEnd) return false;  // 单词不存在
        node.isEnd = false;
        return node.children.size === 0;  // 是否可以删除节点
    }

    const char = word[index];
    const child = node.children.get(char);
    if (!child) return false;

    const shouldDelete = this.deleteHelper(child, word, index + 1);

    if (shouldDelete) {
        node.children.delete(char);
        return !node.isEnd && node.children.size === 0;
    }

    return false;
}
```

**Q3: 如何实现通配符搜索？**
A: 使用DFS：
```typescript
searchWithWildcard(word: string): boolean {
    return this.dfs(this.root, word, 0);
}

private dfs(node: TrieNode, word: string, index: number): boolean {
    if (index === word.length) {
        return node.isEnd;
    }

    const char = word[index];

    if (char === '.') {
        // 尝试所有子节点
        for (const child of node.children.values()) {
            if (this.dfs(child, word, index + 1)) {
                return true;
            }
        }
        return false;
    } else {
        const child = node.children.get(char);
        if (!child) return false;
        return this.dfs(child, word, index + 1);
    }
}
```

**Q4: 空间优化方法？**
A:
1. **路径压缩**：连续单子节点压缩成字符串
2. **双数组Trie**：适合静态字典
3. **哈希Trie**：动态字典，节省空间

**Q5: Trie的实际应用？**
A:
- **搜索引擎**：自动补全
- **拼写检查**：前缀匹配
- **IP路由**：最长前缀匹配
- **输入法**：联想词
- **基因序列**：模式匹配

---

## 复杂度分析

### 时间复杂度

| 操作 | 复杂度 | 说明 |
|------|--------|------|
| insert | O(m) | m是单词长度 |
| search | O(m) | m是单词长度 |
| startsWith | O(m) | m是前缀长度 |

### 空间复杂度

**最坏情况：** O(ALPHABET_SIZE × N × M)
- ALPHABET_SIZE：字符集大小（26）
- N：单词数量
- M：平均单词长度

**实际情况：** 由于前缀共享，空间会小很多

---

## Map vs 数组对比

| 特性 | Map实现 | 数组实现 |
|------|---------|---------|
| 空间效率 | 高（只存在的字符） | 低（26个位置） |
| 访问速度 | 稍慢（哈希） | 快（直接索引） |
| 适用场景 | 字符集大/稀疏 | 字符集小/密集 |
| 代码复杂度 | 简单 | 需要字符转换 |

**推荐：**
- 仅小写字母：用数组
- 包含大小写/数字/特殊字符：用Map

---

## 边界情况

### 情况1：空字符串

```
insert("")：需要特殊处理
root.isEnd = true

search("")：返回root.isEnd
```

### 情况2：单字符

```
insert("a")：
root → a (isEnd=true)

search("a")：true
startsWith("a")：true
```

### 情况3：前缀包含

```
insert("app")
insert("apple")

search("app")：true（不会被apple影响）
search("apple")：true
startsWith("app")：true
```

### 情况4：重复插入

```
insert("apple")
insert("apple")

树不变，只设置isEnd=true（幂等操作）
```

---

## 相关题目

- 211. 添加与搜索单词 - 数据结构设计（中等）- 支持'.'通配符
- 212. 单词搜索 II（困难）- Trie + 回溯
- 648. 单词替换（中等）- 前缀匹配
- 677. 键值映射（中等）- Trie变体
- 720. 词典中最长的单词（简单）
- 1268. 搜索推荐系统（中等）- 自动补全

---

## 总结

**关键点：**
1. Trie是多叉树，用于高效字符串检索
2. 每个节点代表一个字符，路径代表字符串
3. isEnd标记区分前缀和完整单词
4. 时间复杂度只与字符串长度相关

**面试建议：**
- 先画图解释Trie结构
- 说明isEnd的作用
- 解释search和startsWith的区别
- 讨论Map vs 数组的选择
- 可以提及实际应用场景

**易错点：**
- 忘记标记isEnd
- search时没有检查isEnd
- 字符转索引时计算错误
- 空字符串的处理
- 节点创建时机（insert vs search）

**优化方向：**
- 路径压缩（减少节点数）
- 懒删除（标记删除而不是真删除）
- 支持通配符搜索
- 统计词频（每个节点加计数器）
