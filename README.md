# 你的電腦科學教育中遺失的一學期

[![建置狀態](https://github.com/missing-semester/missing-semester/actions/workflows/build.yml/badge.svg)](https://github.com/missing-semester/missing-semester/actions/workflows/build.yml) [![連結檢查狀態](https://github.com/missing-semester/missing-semester/actions/workflows/links.yml/badge.svg)](https://github.com/missing-semester/missing-semester/actions/workflows/links.yml)

這是 [The Missing Semester of Your CS Education](https://missing.csail.mit.edu/) 課程的網站！

非常歡迎對此 REPO 貢獻！如果你有要修改的內容，或想新增新內容，請開一個 issue 或送出 pull request。

## 開發

若要在本機建置並預覽網站，請執行：

```bash
bundle exec jekyll serve -w
```

如果你想在 Docker 容器中開發網站（例如：避免在本機安裝 Ruby 與相關相依套件），請執行：

```bash
docker-compose up --build
```

接著在你的電腦開啟 <http://localhost:4000> 就可以看到網站。當你修改檔案時，Jekyll 會自動重新建置網站。

## 翻譯進度表

| 講座 | 翻譯者 | 校對者 | 是否完成 |
| --- | --- | --- | --- |
| 課程總覽＋Shell 入門 | @AndreaFan123 | 尚未校對 | - [ ] |
| 命令列環境 | @AndreaFan123 | 尚未校對 | ✅ |
| 開發環境與工具 | @AndreaFan123 | 尚未校對 | ✅ |
| 版本控制與 Git | @AndreaFan123 | 尚未校對 | ✅ |
| 除錯與效能分析 | @AndreaFan123 | 尚未校對 | ✅ |
| 程式碼品質 | @AndreaFan123 | 尚未校對 | ✅ |
| 代理式程式開發 | @AndreaFan123 | 尚未校對 | ✅ |
| 不只程式碼 | @AndreaFan123 | 尚未校對 | ✅ |
| 程式封裝與發布 | @AndreaFan123 | 尚未校對 | ✅ |

## 授權條款

本課程的所有內容（包含網站原始碼、講義、習題與課程影片）皆採用 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)。如需了解更多關於貢獻或翻譯的資訊，請參考[這裡](https://missing.csail.mit.edu/license)。
