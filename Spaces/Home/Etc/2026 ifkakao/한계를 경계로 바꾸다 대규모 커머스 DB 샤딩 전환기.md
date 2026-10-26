---
aliases:
  - "한계를 경계로 바꾸다: 대규모 커머스 DB 샤딩 전환기"
---
Hot DB에 read/write

Cold DB에 read only
중간에 CDC 연동해서 스트리밍

## 목표
용량 확보, 트래픽 ? , 유연한 구조

## 샤딩 아키텍처 설계 
consitent hashing ring vs modular

논리 샤드수에 ㄸK라 테이블 크기가 분산
물리 DB 수에 따라 전체 용량이 확장



발표자료 다시 봐야할 듯...
![[IMG-20261014144625302.png]]