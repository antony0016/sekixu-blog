+++
title = '在 csv file 中寫入中文資料'
date = 2024-11-18T15:00:00+08:00
draft = false
categories = ['Backend']
tags = ['Python', 'Encoding', 'CSV']
aliases = ['/posts/python/2024_11_18_csv_file_with_chinese_content/', '/posts/python/2024_11_18_csv_file_with_chinese_content/index.html']
+++

## 前言

原先要為了不同作業系統處理不同編碼，但原本 UTF-8 就應該可以被所有作業系統給讀取，於是研究了一下並把注意事項做成筆記！

## UTF-8 vs UTF-8-sig

兩個都是 UTF-8 的編碼，不過 UTF-8-sig(UTF-8 with signature) 是在檔案開頭帶了 `\ufeff` 的BOM(Byte Order Mark)，定義一個，讓打開他的檔案知道這是個 UTF-8 編碼的檔案。

## 開啟方式的解決方案

在開啟檔案的時候，直接透過記事本或是 VSCode 其實是可以讀取到正確的內容的，但是在 Excel 中打開會是亂碼，那是因為 Excel 沒有讀到 BOM(Byte Order Mark)，所以沒辦法正確的解碼中文字，那麼就會顯示亂碼了。

使用記事本或是 VSCode 就會用正確的編碼開啟了。

## 寫入方式的解決方案

剛剛提到前面少了 BOM 讓 Excel 不知道怎麼開，那就在開頭加上 BOM，要在檔案中寫入 BOM 的方法有兩種，一個是使用 UTF-8-sig，另一個就是自己寫 BOM 進去檔案了，這樣就可以讓編輯器知道要用 UTF-8 來解碼這個檔案！

1. 使用 UTF-8-sig

```python
with open(path, 'w', newline='', encoding='utf-8-sig') as csv_file:
    csv_file.write(data)
```

1. 寫入 BOM

```python
with open(path, 'w', newline='', encoding='utf-8') as csv_file:
    # 寫入 UTF-8 BOM
    csv_file.write(u'\ufeff'.encode('utf8'))
    csv_file.write(data)
```

## 注意事項

寫入方式的解決方案都是在檔案開頭帶了一個 BOM，讓編輯器看得懂檔案，不過在下載的檔案需要被匯入系統的場景就需要特別注意，因為對於程式來說 BOM 就是多了一個字元在前面，匯入時就需要處理掉那個字元，不然就會導致讀取錯誤！

## 結語

平常很少碰到檔案讀寫，第一次知道 EXCEL 讀取 CSV 時，需要透過這個去辨識檔案編碼，希望這個文章可以幫助到其他遇到這個問題的人。