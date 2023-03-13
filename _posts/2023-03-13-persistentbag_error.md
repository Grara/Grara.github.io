---
layout: single
classes: wide
title:  "JPA PersistentBag 불변리스트 관련 에러"
categories:
  - spring-data-jpa
sidebar_main: true
share: false
---
## 에러 발견
- 당근마켓 클론코딩 팀프로젝트 진행 도중 거래글의 엔티티에 @PrePersist를 이용한 메소드가 있었는데 해당 부분에서 에러 발생

```java
@Entity
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class SalesPost {

    @ElementCollection(fetch = FetchType.LAZY)
    @Column(name = "image_url")
    private List<String> imageUrls = new ArrayList<>(); //상품 이미지 url리스트

    @PrePersist //영속화 되기전에 실행할 로직
    public void setting(){
        /*
        이미지가url이 하나도 없을 경우 url로 "없음" 추가
        */
        if(getImageUrls() == null)
            setImageUrls(List.of("없음"));
    }
}
```

- 이상하게 테스트코드에서 수동으로 "없음" 을 추가해주면 에러가 발생하지않음

```java
post.getImageUrls().add("없음");
```

- 에러메시지를 읽어보니 PersistentBag이라고 나온 부분이 있어서 리스트쪽에 뭔가 문제가 있다고 추정하고 검색을 해봄

## 원인
- 지금까지 ArrayList와 List.of()로 생성되는 리스트가 같은 리스트인줄 알았는데 알고보니 ArrayList는 가변 리스트고 List.of()로 생성되는 리스트는 불변 리스트였다.
- PersistentBag은 원본 컬렉션을 감싸는 하이버네이트의 래퍼 컬렉션타입이다.
- PersitentBag은 내부적으로 ArrayList를 사용하며, 불변 리스트를 감쌀 수 없기때문에 에러가 발생한다.

## 해결법
- @PrePersist부분에서 List.of를 new ArrayList로 변경했다.

```java
@PrePersist //영속화 되기전에 실행할 로직
    public void setting(){
        /*
        이미지가url이 하나도 없을 경우 url로 "없음" 추가
        */
        if(getImageUrls() == null){
            setImageUrls(new ArrayList<>());
            getImageUrls.add("없음");
        }
    }
```