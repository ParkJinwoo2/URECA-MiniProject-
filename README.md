# 유레카에서 진행한 첫번째 개인 미니프로젝트

### 랭킹 관리 시스템 — JDBC + Swing 기반 CRUD 구현  

---

## 테이블 생성 SQL문  

```sql
CREATE TABLE players (
    player_id INT AUTO_INCREMENT PRIMARY KEY,
    player_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE genres (
    genre_id INT AUTO_INCREMENT PRIMARY KEY,
    genre_name VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE games (
    game_id INT AUTO_INCREMENT PRIMARY KEY,
    game_title VARCHAR(100) NOT NULL,
    genre_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_game_genre FOREIGN KEY (genre_id)
        REFERENCES genres(genre_id)
        ON DELETE SET NULL
);

CREATE TABLE game_rankings (
    id INT AUTO_INCREMENT PRIMARY KEY,
    player_id INT NOT NULL,
    game_id INT NOT NULL,
    score INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_ranking_player FOREIGN KEY (player_id)
        REFERENCES players(player_id)
        ON DELETE CASCADE,
    CONSTRAINT fk_ranking_game FOREIGN KEY (game_id)
        REFERENCES games(game_id)
        ON DELETE CASCADE,
    CONSTRAINT unique_player_game UNIQUE (player_id, game_id)
);

```

## ERD & 프로젝트 구조

<p align="center">
  <img width="454" height="500" alt="image" src="https://github.com/user-attachments/assets/96ac4fef-e1d8-4004-9360-e869b674c751" />
  <img width="454" height="500" alt="image" src="https://github.com/user-attachments/assets/cc511490-1c1f-4038-9eeb-0b442a1ff2eb" />

</p>  

## 데이터 흐름  
사용자가 Swing 화면(View)에서 입력/버튼 클릭  
Controller가 이벤트를 받아 Service 호출  
Service가 로직 검증 후 DAO 호출  
DAO가 MySQL에 SQL 실행 → DTO 객체 반환  
View -> 결과 표시  

## 주요 기능 
Player / Game / Ranking CRUD 기능  
플레이어별 기록 조회 및 수정  
전체 조회 및 검색 기능  
랭킹 Top10 조회  
FK + CASCADE를 통한 DB 무결성 유지  
DAO / Service 분리로 객체지향적 구조  
DBManager.java 싱글톤 패턴 적용   

## 참조 무결성 유지
→ 존재하지 않는 플레이어나 게임에 랭킹을 넣을 수 없음.  
## 데이터 일관성 관리
→ games가 삭제될 때, game_rankings를 자동 정리.  

## 실행 화면  

<p align="center">
  <img width="454" height="500" alt="image" src="https://github.com/user-attachments/assets/b013c781-7b57-4350-b3df-12dbd01d5fef" />
  <img width="454" height="500" alt="image" src="https://github.com/user-attachments/assets/0a8187c7-10b7-4d76-bfff-4edc1ea1f479" />
</p>

<p align="center">
  <img width="454" height="500" alt="image" src="https://github.com/user-attachments/assets/cecb8d2e-299e-4a0e-8f09-2843e1f7543e" />
  <img width="454" height="500" alt="image" src="https://github.com/user-attachments/assets/caabcd2e-1cfd-448d-a565-8828d47eb676" />
</p>


## Game20 인 게임 -> 랭킹 TOP10 조회
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/682038d4-5a09-4173-9213-e80067715fa7" />  

## 인덱스 기반 쿼리 최적화 
1000건의 데이터 -> 100만건 데이터 
=> EXPALAIN 결과를 확인해보고

<img width="454" height="151" alt="image" src="https://github.com/user-attachments/assets/294857ae-1a21-4667-a00d-eb7924456741" />


```sql
CREATE INDEX idx_score ON game_rankings(score);
```

<img width="1696" height="216" alt="image" src="https://github.com/user-attachments/assets/4c9eb3fe-a405-437c-beca-8dc6f864703b" />  

인덱스 추가 → type= ALL -> range scan 으로 바뀌고, rows **998700 ->189106**  수치 확 줄어듦 → 성능 체감

<img width="1681" height="471" alt="image" src="https://github.com/user-attachments/assets/a0866f07-282b-4bf9-87dd-c37132b073b7" />

## 프로젝트 요약
| 항목           | 기술                                 |
| :----------- | :--------------------------------- |
| **DB**       | MySQL (정규화, FK, UNIQUE, INDEX)     |
| **Backend**  | Java (JDBC, DAO/DTO/Service 계층 분리) |
| **Frontend** | Java Swing (테이블, 버튼 이벤트 기반)        |
| **디자인 패턴**   | Singleton (DBManager)              |
| **최적화 포인트**  | 인덱스, 복합 UNIQUE, FK Cascade         |
| **ERD 완성도**  | 3NF (Third Normal Form) 수준 정규화     |

