---
sidebar:
  nav: "tech"
---
Rober C. Martin 의 'Clean Architecture' 내용 요약.

## 사례연구
  - 기간이 증가할 수록 엔지니어링 직원 수 증가 한다.
  - 제품 라이프 사이클이 증가 할수록 제품 크기(KLOC) 커진다.
  - 제품 라이프 사이클이 증가 할수록 코드 라인당 비용이 높아진다.
  - 출시에 다가가면 생산성은 급속히 떨어진다.
  - 출시 기간이 미뤄 지면 비용은 급속히 증가
  - TDD 적용하면 Interation 될 수록 작업 시간이 줄어 든다.  

## 2가지 가치
1. 행위
  - 이해관계자가 프로그래머를 고용하는 이유는 기계가 수익 창출 혹은 비용 절감
  - 프로그래머는 이해관계자의 기능 명세서나 요구사항 문서를 구체화 하도록 돕는다.
2. 아키텍처
  - 변경 용이: 변경의 어려움은 'Scope'에 비례하지만 'Shape'와는 무관
  - 우선 순위: 중요/긴급, 중요/긴급X, 중요X/긴급, 중요X/긴급X  

## 프로그래밍 패러다임
  - 패러다임
    - 구조적 : 1968년 Edsger Wybe Dijkstra, 제어흐름의 직접적인 전환에 대한 규칙 부과
    - 객체지향 : 1966년 Ole Johan Dahl, Kristen Nygaard, 제어흐름의 간접적인 전환에 대한 규칙 부과
    - 함수형: 1930년대 Alonzo Church, Lamda 계산법, 1958년 John McCarthy, LISP 할당문에 대한 규칙을 부과  

  - 객체지향 프로그래밍
    - '캡슐화', '상속', '다형성' 개념은 적절하게 조합한다.
    - OO언어는 최소한 3가지 요소를 반드시 지원해야 한다.
    - 다형성
      - 의존성 역전(Dependency Inversion)
        - A -> Interface B <- Implementation B
      - 플러그 인(Plug-In)
        - UI -> Business Rules <- Database : UI와 Database가 업무 규칙의 플러그 인이 된다.

  - 함수형 프로그래밍      
    - 예) Java Code 과 함수형 Code
      - Java Code
        ```
        public class Squit {
          public static void main(String args[]){
              for(int i=0;i<10;i++)
                  System.out.println(i*i);
          }
        }
        ```   
      - 함수형 Code
        ```
         (println (take 25 (map (fn [x] (* x x)) (range) )))
        ```
        1. println, take, map, range 는 모두 함수 ()안에 함수를 넣어서 함수를 호출  
        1. (fn [x] (* x x)) 는 '익명 함수', '곱셈 함수'를 호출 하면서 '입력 인자'를 두번 전달
        1. 가장 안쪽 함수 부터 호출
        1. range 함수는 0부터 시작 해서 끝이 없는 정수 리스트 반환
        1. 반환된 정수 리스트는 map 함수로 전달, 각 정수에 제곱을 계산하는 익명 함수 호출
        1. 제곱된 리스트는 take 함수로 전달, 앞에 10개 까지 항목으로 구성된 새로운 리스트 반환
        1. 앞의 10개 정수에 대한 제곱 값으로 구성된 리스트를 println 함수에 입력 하여 출력

    - 불변성과 아키텍처 : 가변으로 발생하는 문제는 경합(race), 교착(deadlock), 동시(concurrent)이다.
    - 가변성의 분리 : 가변 컴포넌트와 불변 컴포넌트 분리
    - 이벤트 소싱 
      - 트랜잭션을 저장, 상태가 필요하면 상태의 시작점부터 모든 트랜잭션을 처리
      - CRUD가 아니라 CR만 발행, 완전한 불변성, 완전한 함수형, 예) 은행 잔고 조회 
    
## 설계원칙 (SOLID; SRP, OCP, LSP, ISP, DIP)
 - SRP(Single Responsibility Principle)
   - 개념
     - 소프트웨어 모듈은 변경의 이유가 오직 하나 뿐 이어야 한다.
     - 변경을 요청하는 'actor'은 1명이 아닌 집단
     - 하나의 모듈은 오직 하나의 'actor'에 대해서만 책임져야 한다.
     - Conway's law: 소프트웨어가 가질 수 있는 최적의 구조는 시스템을 만드는 조직의 사회적 구조에 커다란 영향을 받는다.
   - 징후
     - 중복, 병합
   - 해결
     - 클래서 분리, Facade
   - Method와 Class 수준의 원칙, Component 관점에서는 Common Closure Principle, 
     아키텍처 관점에서는 Boundary를 생성을 책임지는 Axis of Change가 된다. 
 
 - OCP(Open-Close Principle)
   - 1980년대 Bertrand Meyer, 기존 코드 수정 보다는 신규 추가로 코드로 확장

 - LSP(Liskov Substitution Principle)
   - 1988년 Barbara Liskov, 하위 타입에 관한 원칙, 구성 요소는 상호 치환 가능
   
 - ISP(Interface Segregation Principle)
   - 사용 하지 않는 것에 의존 하면 안됨

 - DIP(Dependency Inversion Principle)
   - 고수준 정책을 구현 하는 코드는 저수준 세부 사항을 구현 하는 코드에 절대 의존 하면 안된다.