---
title:  "@ModelAttribute vs @RequestBody (바인딩 실패 시 차이점)"
categories: 스프링
toc: true
toc_sticky: true
toc_label: "Contents"
---

## 문제 상황
- API 개발 도중 **<span style='background-color:#fff5b1'>@RequestBody</span>** 애노테이션을 사용해 객체 변환 도중 <span style="color:red;">바인딩 실패 시</span>   
BindingResult에 에러가 담길 줄 알았지만, 400 에러가 나는 상황이 발생했습니다.
- **@ModelAttribute**나 **@RequestParam**의 경우 바인딩 실패 시 BindingResult에 해당 에러가 담기는 데 반해 @RequestBody를 사용할 경우에는 담기지 않아 의문이 생겼습니다.
- 그래서 @ModelAttribute와 @RequestBody의 동작원리와 위에서 발생한 문제를 해결하는 방법을 정리하려합니다.

