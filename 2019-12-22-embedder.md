---
title: "GraphQL이랑 비슷한 걸 만든 이야기"
date: "2019-12-22"
category: "tech"
cover: https://graphql.org/
tags:
    - GraphQL
---

이번에 회사에서 리셀러를 위한 웹사이트를 개발하는 프로젝트를 했는데, 서버/클라이언트 모두 혼자서 개발했기 때문에 기술적으로 이것저것 자유롭게 시도해볼 수 있는 기회였다. 클라이언트에는 지난 글에서 언급했던 React 상태관리 라이브러리를 적용해서 빠른 시간 안에 기능 개발을 할 수 있었다.

하지만 걸리는 부분이 있었는데, 하나의 화면을 그리기 위해서 굉장히 많은 서버측 API를 호출해야 하고, 그로 인해 여러 API 콜에 대한 상태 관리, 로딩 처리, 에러 처리 등을 신경써야 했다. 자연스레 GraphQL을 떠올렸지만, GraphQL을 적용하기에는 그렇게 규모가 있는 프로젝트가 아니었고, 또 사내에 GraphQL을 도입한 사례가 없어서 확 써버리기에는 부담스럽기도 했다.

그래서 그냥 REST API를 쓰면서도 GraphQL 비슷한 기능을 쓸 수 있게 만들어 보기로 했다

아직은 미완성이지만, 어느 정도 모양새가 잡혔다. 가령 도서 추천 사이트에서,

- 도서 (book)
  - 해당 도서의 저자 (author)
  - 해당 도서의 출판사 (publisher)
    - 해당 출판사의 추천 도서 목록 (recommendations)
  - 해당 도서에 대한 리뷰 목록 (reviews)
    - 해당 리뷰의 글쓴이 (user)
      - 해당 글쓴이의 프로필 (profile)

이런 정보들을 불러와야 하는 상황이라고 해보자. 이 때에 embed 쿼리 스트링에 가져오고 싶은 JSON 경로를 넣어준다.

```
GET /book/1?embed=author,publisher.recommendations,reviews.user.profile
```

그러면 서버에서는 아래와 같은 모양의 응답을 보내준다. (GraphQL이랑 똑같다고 보면 된다.)

```json
{ // book
  "title": "Clean Coder",
  // author
  "author": {
    "name": "Robert C. Martin"
  },
  // publisher
  "publisher": {
    "name": "Prentice Hall",
    // recommendations
    "recommendations": [
      {
        "title": "The C Programming Language"
      }
    ]
  },
  // reviews
  "reviews": [
    {
      "content": "Very helpful!",
      // user
      "user": {
        // profile
        "profile": {
          "name": "Kim Seungha"
        }
      }
    }
  ]
}
```

처음에는 GraphQL에 있는 필드 선택, 필드별 인수같은 기능들도 생각했지만, 만들면서 보니 이런 기능이 없는 편이 REST API와 궁합이 더 잘 맞겠다는 생각이 들었다.

GraphQL에 있는 필드 선택 기능은 overfetch를 없애줌으로써 굉장히 호평받는 기능이지만, 클라이언트 측에서 엔티티 모양이 매번 달라질 수 있으므로 TypeScript를 통한 타입 정의를 어렵게 만든다. GraphQL 쿼리로부터 TypeScript 타입 정의를 자동 생성해주는 도구가 있지만, 이렇게되면 쿼리마다 다른 타입을 쓰게되어 코드 재사용성이 떨어지게 된다. Facebook 같이 엄청나게 많은 트래픽을 견뎌야 하는 경우라면 필드 선택을 세세히 하는 것이 네트워크 비용 절약에 도움이 될 수 있겠으나, 사람값이 더 비싼 스타트업에서는 언제나 일관적인 모양으로 데이터를 받을 수 있으면 좋겠다는 생각을 했다.

또한 GraphQL 쿼리의 필드별로 인수를 넣을 수 있는 기능 역시 과하다는 생각을 했다. REST API에서는 일반적으로 쿼리 스트링을 사용하므로, 이걸 그대로 쓰면 될 거라 생각하고 있다. 개발을 하다가 안쪽에 있는 노드에서 페이지네이션이나 필터링을 필요로 하는 경우가 생길 수 있는데, 이 때에는 그 부분을 별도의 REST API로 만들어서 쓰면 충분할 거라고 생각하고 있다.

또한 GraphQL 서버의 리졸버를 구현하는 방식과는 다른 방식으로 레코드를 가져오도록 만들었다. 예를 들어 has many 관계에서, '부모 레코드의 ID'를 가지고 '자식 레코드 전체'를 가져오는 함수를 제공하도록 만들었다.

```clojure

(def reseller-embedder
  ; 트리 루트의 이름은 reseller이다.
  (-> (graph-root "reseller")
      ; 간선을 등록한다.
      (register-edge
        ; 이 간선은 reseller와 reseller-sale-items를 잇는다.
        ["reseller" "reseller-sale-items"]
        ; reseller는 reseller-sale-items 목록을 갖는다.
        (map->HasManyRelation
           ; 두 엔티티는 reseller-id를 통해 연결된다.
          {:fk                         :reseller-id
           ; reseller-sale-items 레코드의 필드 중 일부분만 JSON에 포함시킨다.
           :select                     [:reseller-id :product-id :discount :postpaid]
           ; *** reseller id를 가지고 reseller-sale-items를 가져오는 함수 ***
           :get-children-by-parent-ids reseller-sale-items-dao/get-reseller-sale-items-by-reseller-ids}))
...
```

GraphQL 서버 측 구현체들은 '부모 레코드 하나'를 가지고 '자식 레코드 하나'를 가져오는 방식으로 resolver를 구현하는 것이 기본이기 때문에, 아무 생각 없이 쓰면 N+1 쿼리 문제가 생기고 덕분에 'GraphQL은 느리다'는 소문(?)이 퍼지게 되었다. 그래서 '모든 부모 레코드'를 가지고 '모든 자식 레코드'를 가져온 다음에, 나중에 관계를 지어주는 방식으로 추가 구현을 해주는 것이 필수다. 이를 위해 [DataLoader](https://github.com/graphql/dataloader) 라이브러리가 공식적으로 제공된다.

위 Clojure 구현체에서는 아예 DataLoader 방식이 기본이 되도록 만들었다. 사용법이 그리 직관적이지는 않지만, 어차피 DataLoader를 사용하는 것이 반강제인 상황에서는 이걸 기본으로 하는 것이 맞겠다는 생각을 했다.
