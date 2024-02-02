# Cryptography


學習如何思考密碼構造的安全性，以及如何將這些知識應用於實際應用中。當一個強大的對手竊聽和干擾通訊時，兩個具有共享秘密密鑰的方可以安全通訊的方式。

檢視許多的協議並分析現有系統中的錯誤。再來討論了讓兩個方生成共享秘密密鑰的公鑰技術。涵蓋相關的數論並討論公鑰加密和基本金鑰交換。


## 基本概念定義

1. **universe U**：在離散機率中，宇宙（以 "U" 表示）指的是有限的元素集合。在許多密碼學情境中，宇宙通常是所有可能的 n 位元字串的集合，表示為 {0, 1}^n。
2. **機率分佈 (P)**：機率分佈是一個函數（以 "P" 表示），它為宇宙 U 中的每個元素分配一個介於 0 和 1 之間的數字。這些數字表示每個元素的機率或權重，分佈中所有權重的總和始終等於 1。
3. **均勻分佈**：均勻分佈將宇宙中的每個元素分配相等的權重。對於大小為 |U| 的宇宙，每個元素都被分配機率 1/|U|。
4. **點分佈**：點分佈將所有權重集中在宇宙中的單個元素（點）上，給予它機率 1，同時將所有其他元素分配機率 0。
5. **事件**：事件是宇宙 U 的一個子集（以 "A" 表示）。事件的機率是該子集中所有元素的權重之和。
6. **聯集界限**：聯集界限是一個簡單的機率不等式。對於兩個事件 A1 和 A2，它們的聯集的機率小於或等於它們各自的機率之和。如果事件是不交集的（沒有共同元素），那麼不等式變為等式。
7. **隨機變數**：隨機變數（以 "X" 表示）是一個函數，將宇宙 U 中的元素映射到可能值的集合（以 "V" 表示）。它表示一個隨機過程，輸出會根據隨機性而變化。
8. **均勻隨機變數**：均勻隨機變數是從宇宙 U 中均勻隨機抽樣的變數。它為 U 中的每個元素分配相等的機率。
9. **隨機算法**：隨機算法是一種算法，它接受一個輸入和隱含生成的隨機輸入（從 U 中抽樣而來）。由於隨機輸入的不同，每次運行算法都會得到不同的輸出，實際上定義了一個表示可能輸出分佈的隨機變數。
10. **範例**：給出一個範例，其中一個隨機算法使用隨機生成的字串作為加密金鑰，對消息 M 進行加密。該算法的輸出是一個隨機變數，表示在給定消息 M 下，使用均勻金鑰進行加密的可能密文的分佈。


## 一次性密鑰

一次性密鑰是一種加密方法，它使用一個與訊息相同長度的秘密密鑰，將訊息與密鑰進行XOR運算來加密訊息。解密時再次使用相同的密鑰來進行XOR運算，可以還原原始訊息。這種方法在理論上非常安全，被稱為"完美安全性"。

然而，問題在於這個密鑰只能使用一次。如果相同的密鑰被用於加密多條消息，則攻擊者可以輕鬆地破解這些訊息。攻擊方式被稱為"雙重一次性密鑰攻擊"，攻擊者可以獲得兩條訊息的密文，並進行XOR運算以獲得這兩條訊息的XOR結果，然後通過分析這個XOR結果來恢復兩條原始訊息。

因此，一次性密鑰或流加密的密鑰絕不應該被多次使用，否則攻擊者可以輕易破解您的訊息。這是一個常見的安全性錯誤，應在設計安全系統時避免。

## 實際應用中使用的不同流加密方法 :

1. **RC4**:
    - RC4是一種於1987年設計的流加密方法
    - 它使用可變大小的種子，例如128位，作為流加密的密鑰
    - RC4將秘密金鑰擴展為2,048位，並將其用作內部狀態
    - 它以簡單的循環方式生成輸出，每次生成一個字節
    - 儘管它很受歡迎，但RC4存在弱點，因此不建議用於新項目。例如，輸出中的某些字節存在偏差，並且隨著時間的推移，序列"00"出現的頻率高於預期
2. **CSS（Content Scrambling System）**:
    - CSS是一種用於加密DVD電影的加密方法
    - 它基於線性反饋移位寄存器（LFSR），設計用於硬體實現
    - CSS使用40位密鑰，這是當時出口法規的限制
    - 對CSS的攻擊涉及攔截加密數據，查找已知前綴，並使用它來恢復LFSR的初始狀態，從而最終解密其餘部分的電影

eStream中的一個更好的加密範例，稱為**Salsa20**：

1. **Salsa20**（來自eStream項目）：
    - Salsa20使用128位或256位密鑰和64位的nonce
    - 它使用名為“H”的函數生成一個長的偽隨機序列
    - 函數H將密鑰、nonce和計數器擴展為64個字節
    - 函數H被重複應用（10次），其輸出與輸入進行加法混合
    - Salsa20旨在供軟體和硬體實現使用，提供了高安全性

Salsa20只是eStream項目核准的流加密方法之一，適用於現代加密需求，具有良好的安全性

## PRG Security Definitions

### 隨機生成器（PRG）

“不可區分於隨機”的概念。一個PRG的目標是生成看起來與真正的隨機數一樣的數據，這種生成的數據應該無法被區分出來。我們使用了“統計測試”的概念來衡量PRG的安全性，統計測試是一個算法，它根據輸入的數據（比如一個位元字符串）來決定該數據是否看起來是隨機的。

統計測試的範例，這些測試可以檢查數據中的不同特徵，例如位元的平衡或連續的零。我們通過計算統計測試的“優勢”來衡量它們的能力，這個優勢告訴我們測試是否能夠區分生成的數據與真正的隨機數據。

一個安全的PRG必然是不可預測的。這意味著給定生成的前綴部分，無法預測下一位的輸出。這個結果是基於對PRG安全性的定義和統計測試的能力進行的。

Yao's Theorem，如果一個生成器是不可預測的，則它必然是安全的。換句話說，不可預測性是PRG安全性的一個必要條件。我們也討論了一個應用，即如果給定生成的輸出的最後位元，可以預測前面的位元，那麼生成器就是不安全的。

計算不可區分性的符號，用於衡量兩個分佈是否可以在多項式時間內被區分開。這個符號對於定義加密算法的安全性非常有用。

## 區塊密碼系統

一種加密機制，由加密算法E和解密算法D組成。這些算法都需要一個密鑰K。區塊密碼將N位元的明文轉換為同樣數量的N位元密文。典型的區塊密碼有Triple-DES和AES。

1. **Triple-DES**：使用64位元的區塊大小，密鑰長度為168位元。它將64位元的明文映射到64位元的密文
2. **AES**：使用128位元的區塊大小，密鑰長度可以是128、192或256位元。AES將128位元的明文映射到128位元的密文

這些區塊密碼通常是通過迭代構建的，意味著明文會被多次加密，每次使用不同的子密鑰。這些子密鑰來自於原始密鑰K的擴展。

一個區塊密碼的安全性可以透過其對應於 **偽隨機函數（PRF）** 或 **偽隨機置換（PRP）** 的性質來衡量。PRF和PRP都應該看起來像是隨機的，也就是說，對於一個沒有密鑰的攻擊者來說，他們不能區分加密後的結果和隨機數據的區別。

最後，區塊密碼除了用於加密，還可以用於確保數據的完整性和認證等多種應用。接下來的部分將會深入探討這些區塊密碼的具體構造和安全性。

1. **偽隨機函數 (PRF)**：
    - 定義於一個密鑰空間、輸入空間和輸出空間
    - 是一種函數，根據給定的密鑰和輸入產生輸出
    - 重點在於，對於不知道密鑰的攻擊者來說，PRF的輸出應該與真正隨機函數的輸出無法區分
    - PRF不要求函數是可逆的，只要求給定密鑰和輸入時可以有效計算輸出
2. **偽隨機置換 (PRP)**：
    - 也是定義在一個密鑰空間上，但它的輸入和輸出空間是相同的集合X
    - PRP不僅要求輸出看起來隨機，還要求函數是一一對應的，即每個輸入都有唯一的輸出，且每個輸出都來自唯一的輸入
    - 這意味著PRP是可逆的，存在一個有效的解密算法
    - 在密碼學中，區塊密碼通常被看作是PRP，因為它們將固定大小的輸入區塊映射到固定大小的輸出區塊，且每個輸出區塊都有一個唯一的對應輸入區塊

## DES（資料加密標準）

一種經典的區塊密碼系統，起源於1970年代。其基於一種稱為 Feistel 網絡的結構，透過多輪的轉換將明文轉變為密文。每一輪都使用一個轉換函數和一個子密鑰對資料進行加密。

DES 的關鍵特點包括：

1. **64位元的區塊大小**：明文和密文都是64位元
2. **56位元的密鑰長度**：儘管密鑰總長度為64位元，但其中8位元用於奇偶校驗，實際有效的密鑰長度是56位元
3. **16輪加密過程**：DES將明文經過16輪的 Feistel 網絡處理，每輪使用不同的子密鑰

Feistel 網絡允許加密和解密過程非常相似，只需將子密鑰的使用順序倒過來即可。每輪包括位元置換和一個非線性轉換（S盒）。S盒是DES安全性的核心，負責確保加密過程有足夠的非線性特性。

DES由於其56位元的密鑰長度在現代被認為太短，容易受到 **窮舉攻擊（即嘗試每一個可能的密鑰）** 。這導致了後來 **AES（高級加密標準）** 的誕生，AES有更長的密鑰並且被認為更安全。
