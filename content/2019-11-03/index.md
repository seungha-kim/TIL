---
title: "Elixir + Phoenix + Ecto"
date: "2019-11-03"
category: "tech"
cover: https://phoenixframework.org
tags:
    - Elixir
    - Phoenix
---

첫 회사에서 Ruby on Rails를 쓸 때는 이게 뭐가 좋은 건지 몰랐었는데, 그 뒤로 Flask, Express, Compojure 등 'microframework' 스타일의 웹 프레임워크를 동료 개발자들과 써오면서, 프로젝트 설계와 코드 컨벤션을 높은 품질로 유지하는 것이 얼마나 어려운 일인지를 배웠다. 일단 이 놈들은 얇은 웹 프레임워크 위에 직접 차곡차곡 레이어를 쌓아올리는 식으로 개발을 해야 하는데, 이게 여러가지 이유로 잘 되기가 어렵더라..

그래서 요즘은, 소프트웨어 설계에 시간과 에너지를 투자하기 힘든 환경이라면, 양질의 ORM과 'convention over configuration' 철학으로 편하게 개발할 수 있는 환경을 제공해주는 RoR 같은 프레임워크를 쓰는 것이 맞지 않나 하는 생각을 하게 됐다. 뻔하지만 반복되는 문제들에 대해 미리 갖추어진 것이 많은 환경이라면, 의사결정할 것들이 크게 줄어든다.

Phoenix는 Elixir 언어를 만든 회사인 Plataformatec에서 직접 내놓은 웹 프레임워크로, 같은 곳에서 만든 Ecto라는 DB 라이브러리와 같이 쓰면 Ruby on Rails 만큼의 편안함을 가져다준다. Elixir는 Ruby의 많은 부분을 모방한 언어이고, 이것으로 만든 Phoenix 역시 RoR과 많은 부분이 닮아있다.

- MVC 패턴을 쉽게 구현할 수 있게 해 주는 매크로 기반 DSL
- Sinatra, Express 스타일의 미들웨어
- Resourceful routing
- Project/code generator CLI
- Database migration CLI
- REPL을 통한 디버깅

이 모든 것들이 내장이고, CLI 커맨드 하나로 생성된 프로젝트에서 위 기능들을 즉시 사용할 수 있다. 여기에 Elixir 만이 가지는 특징들은 덤이다.

- Ruby 대비 [높은 성능](https://medium.com/@elviovicosa/phoenix-vs-rails-benchmark-2019-f0e68336d557)
- 실시간 웹 하기 좋음
- 함수형 -> 테스트하기 쉬움
