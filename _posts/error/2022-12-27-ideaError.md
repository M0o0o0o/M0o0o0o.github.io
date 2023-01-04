---
title: "Mac OS zsh에서 'idea .' 명령어 등록하기"  
categories: etc
toc: true
toc_sticky: true
toc_label: "Contents"
---

새로운 맥북에서 "idea ." 명령어가 동작하지 않아 여러 글을 찾다가 해결해서 공유합니다.

- 터미널에서 vi ~/.zshrc 실행
- alias idea='open -a "ls -dt /Applications/IntelliJ\\ IDEA*|head -1"'
  - 코드를 작성하고 저장 후 종료
- "source ~/.zshrc" 실행!  