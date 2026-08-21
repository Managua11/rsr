# RSR — „რა სად როდის" აპლიკაცია

**პროექტის გეგმა და მიღებული გადაწყვეტილებები**
ბოლო განახლება: 2026-08-21 · მდებარეობა: `C:\Users\tmanagadze\Desktop\Personal\rsr`

---

## 1. რას ვაკეთებთ

მობაილ აპლიკაცია „რა სად როდის" (ЧГК) სათამაშოდ — კითხვების საკუთარი კატალოგით, პაკეტების ავტორინგით და თამაშის რეჟიმებით.

**მიმდინარე სტატუსი (2026-08-20):** ბექენდი ბაზასთან **დაკავშირებულია და მუშაობს**. `backend/` — Spring Boot 4.1.0, Gradle Groovy DSL; Neon-ის ბაზა მიერთებულია, `V1__init.sql` გაშვებულია და სქემა Neon-ში დგას (11 ცხრილი + `flyway_schema_history`). `./gradlew test` (`contextLoads`) მწვანეა.

ჯერ არ არსებობს: `SecurityConfig` (ე.ი. ყველა endpoint დაბლოკილია), `jjwt`, არცერთი entity / repository / controller. git რეპოზიტორი ინიციალიზებულია (`Initial commit`), მაგრამ `backend/` კვლავ untracked-ია.

---

## 2. ტექნოლოგიური სტეკი

| ფენა | არჩევანი | რატომ |
|---|---|---|
| მობაილი | **Flutter** | ერთი კოდი ორივე პლატფორმაზე; აჯობა React Native-ს და Kotlin Multiplatform-ს |
| ბექენდი | **Java 21 + Spring Boot 4.1.0** | Java უკვე დაყენებულია კომპიუტერზე; Boot 4 — გადაწყვეტილება 2026-08-20, იხ. §2.1 |
| ბილდი | **Gradle (Groovy DSL) + wrapper 9.5.1** | wrapper ნიშნავს, რომ Gradle-ის ცალკე დაყენება არ სჭირდება |
| ბაზა | **Neon** (managed cloud PostgreSQL) | ლოკალურად PostgreSQL-ის დაყენება არ გვინდა; Docker არ არის |
| მიგრაციები | **Flyway** — პირველივე დღიდან | `ddl-auto: update` პროდაქშენში **არასოდეს** |
| ავტორიზაცია | Spring Security + JWT (jjwt) | |
| დოკუმენტაცია | springdoc-openapi (Swagger UI) | |

**გარემო ამ კომპიუტერზე:** Java 21 ✅, Node 24 ✅, Docker ❌, Gradle/Maven CLI ❌ (wrapper გვაქვს), Flutter SDK ❌, PostgreSQL ❌

### 2.1 რატომ Spring Boot 4, და არა 3.x

Initializr-მა 4.1.0 დააგენერირა (გეგმაში 3.x ეწერა). **გადაწყვეტილება — ვრჩებით 4-ზე:** ახალი პროექტია, პროდაქშენში არაფერია, მიგრაცია მაინც მოგვიწევდა.

რასაც ეს ნიშნავს პრაქტიკაში:

- **starter-ების სახელები შეიცვალა.** `spring-boot-starter-web` → **`spring-boot-starter-webmvc`**; Flyway ცალკე starter-ია; ტესტებისთვის თითო მოდულს თავისი `-test` starter აქვს (`spring-boot-starter-data-jpa-test` და ა.შ.). **Boot 3-ის tutorial-ები პირდაპირ არ იმუშავებს** — ეს იქნება მთავარი ხახუნის წყარო.
- **Spring Security 7** მოყვება — კონფიგურაციის DSL შეცვლილია, ძველი მაგალითები არ გამოდგება.
- **მესამე მხარის ბიბლიოთეკები:** `jjwt` Spring-ზე დამოკიდებული არაა → იმუშავებს. `springdoc-openapi` კი Spring-ის შიგნეულობას ეყრდნობა — Boot 4-თან თავსებადი ვერსია ცალკე შესამოწმებელია. **თუ არ აეწყო, Swagger UI გადაიდოს** — ბლოკერი არაა.

---

## 3. ეტაპები

1. **ეტაპი 1 — solo / ასინქრონული MVP** — თამაში მარტო ან ასინქრონულად; კითხვების ავტორინგი; საჯარო კატალოგი.
2. **ეტაპი 2 — realtime multiplayer** — WebSocket/STOMP, გუნდები, ცოცხალი ოთახები.

გადაწყვეტილება იყო „ორივე — ეტაპობრივად", ამიტომ **ეტაპი 1-ის არქიტექტურა ეტაპი 2-ს არ უნდა გადაეღობოს:**

- თამაშის ლოგიკა **`GameEngine`**-ში, ტრანსპორტისგან დამოუკიდებლად — რომ ეტაპი 2-მა WebSocket უბრალოდ ზემოდან დაადოს და REST კონტროლერები არ გადაიწეროს.
- მდგომარეობასთან წვდომა **`GameStateRepository`** ინტერფეისის უკან — რომ აქტიური ოთახებისთვის მოგვიანებით Redis ჩავსვათ ბაზის ნაცვლად.
- `teams` / `team_members` ცხრილები და nullable `game_sessions.team_id` **პირველივე Flyway მიგრაციაში** შევიდეს, თუნდაც ეტაპი 1-ში ცარიელი დარჩეს.

---

## 4. კონტენტის სტრატეგია

### 4.1 db.chgk.info — უარყოფილია

მონაცემთა ბაზის უდიდესი ნაწილი **CC BY-NC-ND** ლიცენზიითაა:

- **ND** — ქართული თარგმანი წარმოებული ნაწარმოებია და დაუშვებელია;
- **NC** — მონეტიზაცია გამორიცხულია.

მხოლოდ რამდენიმე ავტორს აქვს უფრო თავისუფალი პირობები (ერთი CC BY-ND, სამი CC BY-NC-SA) და არცერთი კომბინაცია არ იძლევა „თარგმნილი ქართულად **და** კომერციული"-ს. ცალკე პრობლემა: ЧГК კითხვების დიდი ნაწილი რუსულ სიტყვათა თამაშს ეყრდნობა და თარგმანს საერთოდ ვერ იტანს.

**შედეგი:** კონტენტი არის ორიგინალური — ავტორი თვითონ (promodev@crococloud.pl) წერს კითხვებს, პლუს იუზერების მიერ შექმნილი კონტენტი.

### 4.2 კითხვა არის მთავარი ერთეული, პაკეტი — თემატური ხედი

ეს გადაწყვეტილება შეგნებულად აირჩა უფრო მარტივი „პაკეტი ფლობს კითხვას" მოდელის ნაცვლად, რადგან **პაკეტს კონკრეტული თემა უნდა ჰქონდეს**, კითხვა კი შეიძლება არცერთ პაკეტში არ იყოს.

- **`questions`** — საკუთარი `visibility` (PRIVATE | PUBLIC) და `status` (DRAFT → PENDING_REVIEW → PUBLISHED | REJECTED | ARCHIVED). საჯარო კატალოგი დამოუკიდებელია.
- **`packages`** — საკუთარი `visibility` (PRIVATE | UNLISTED | PUBLIC), `status`, `theme`, `share_code`.
- **`package_questions`** — კავშირი, `position` და `tour_number` ველებით.

**ხილვადობის წესი:** აკრძალულია მხოლოდ ერთი კომბინაცია — **PRIVATE კითხვა PUBLIC პაკეტში**. შემოწმდეს პაკეტის გამოქვეყნებისას, არა ყოველი კითხვის დამატებისას. PUBLIC კითხვა PRIVATE პაკეტში — სრულიად დასაშვებია.

### 4.3 მოდერაცია

- მოდერაცია **კითხვის** დონეზეა — ის არის კონტენტის ატომური ერთეული.
- პაკეტი, რომელიც მხოლოდ უკვე დამტკიცებული საჯარო კითხვებისგან შედგება, ახალ კონტენტს არ ამატებს → მისი შემოწმება მსუბუქია (მხოლოდ მეტამონაცემები) და შეიძლება ავტომატურად გამოქვეყნდეს.
- **პირად პაკეტებს მოდერაცია არ სჭირდება** — ეს გეიტი პროდუქტის მთავარ ღირებულებას მოკლავდა (ადამიანი თავისი საღამოსთვის ამზადებს პაკეტს).
- საჯარო კატალოგი — **ეტაპი 1-იდანვე** (მომხმარებლის გადაწყვეტილება; ალტერნატივა release 2-ზე გადადება იყო, მოდერაციის დატვირთვის გამო). ეს ამატებს: `reports` ცხრილს, მოდერატორის endpoint-ებს, `PENDING_REVIEW` ნაკადს და კატალოგის ძებნა/ფილტრაციას.

### 4.4 კონტენტის წესები, რომლებიც არ უნდა დაირღვეს

- **PUBLIC → PRIVATE გადაყვანა აკრძალულია** კითხვაზე, რომელიც სხვის პაკეტებში შეიძლება უკვე იყოს. ამის ნაცვლად — **ARCHIVED**: კატალოგში აღარ ჩანს, ახალ პაკეტში ვეღარ დაემატება, არსებულებში კი კვლავ თამაშდება.
- **`session_answers` უნდა ინახავდეს კითხვის ტექსტისა და პასუხის snapshot-ს** თამაშის მომენტისთვის — რომ გამოქვეყნებული კითხვის შემდგომმა რედაქტირებამ ჩაწერილი შედეგები არ გააუქმოს.
- **`questions.author_id` ჩანდეს** თამაშისას ან შედეგების ეკრანზე — ავტორობის კრედიტი ერთადერთი სტიმულია საჯარო კატალოგში გამოსაქვეყნებლად, კატალოგი კი კონტენტის ზრდის მთავარი ძრავია.

### 4.5 კითხვის რეალური ფორმა

ველები უნდა ფარავდეს ნამდვილ ЧГК სტრუქტურას:

- раздатка (handout) — ტექსტი და/ან მედია
- დამატებითი მისაღები პასუხები (accepted alternatives)
- კომენტარი, წყარო
- **DUPLET / BLITZ** ქვე-კითხვები — `question_parts` ცხრილით

### 4.6 ავტორინგი — ეტაპი 1-ის ნაწილია

უამისოდ კონტენტი საერთოდ არ არსებობს. ეს დაახლოებით აორმაგებს ავტორის მხარის UI-ს (~4-5 დამატებითი Flutter ეკრანი: ცალკე კითხვის რედაქტირება, ცალკე პაკეტის აწყობა).

**აუცილებელია bulk-იმპორტიორი** სტანდარტული ЧГК ტექსტური ფორმატისთვის (`Вопрос / Ответ / Комментарий / Источник`) — ასობით კითხვის ფორმით შეყვანა რეალური არ არის.

---

## 5. ცნობილი რთული ადგილი — პასუხის შემოწმება

ავტომატური fuzzy matching ЧГК-ში საიმედოდ არ მუშაობს. **MVP-ში თვით-შეფასება:** მოთამაშე ხედავს სწორ პასუხს და თვითონ აღნიშნავს „სწორი ვიყავი?".

---

## 6. UGC და აპლიკაციის მაღაზიები

მომხმარებლის შექმნილი კონტენტი App Store / Google Play-ს მოთხოვნებს ააქტიურებს. მათ გარეშე აპლიკაციას უარს ეტყვიან:

- აპლიკაციაში ჩაშენებული საჩივრის (report) მექანიზმი
- მომხმარებლის დაბლოკვის შესაძლებლობა
- გამოქვეყნებული მოდერაციის პოლიტიკა
- ToS, რომელიც იუზერის კონტენტზე ლიცენზიას არეგულირებს

---

## 7. სად ვართ — ბექენდი ბაზასთან დაკავშირებულია

### 7.1 რა შეიქმნა (2026-08-20)

`backend/` საქაღალდე, Spring Initializr-ით IntelliJ-დან:

```
rootProject.name = 'backend'      group = ge.rsr       Java 21 toolchain
Spring Boot 4.1.0                 Gradle Groovy DSL, wrapper 9.5.1

backend/src/main/java/ge/rsr/BackendApplication.java
backend/src/main/resources/application.properties   (მხოლოდ spring.application.name)
backend/src/test/java/ge/rsr/BackendApplicationTests.java
```

უკვე ჩაწერილი dependency-ები: `actuator`, `data-jpa`, `flyway`, `security`, `validation`, `webmvc`, `flyway-database-postgresql`, `postgresql` (runtimeOnly) + შესაბამისი `-test` starter-ები.

> - `rootProject.name` არის **`backend`**, კლასი — `BackendApplication`. თუ `rsr` გვირჩევნია, გადარქმევა ახლა ყველაზე იაფია.
> - ორი IntelliJ პროექტია ჩალაგებული — `rsr/.idea` და `rsr/backend/.idea`. Flutter-ის დამატებისას root-ად `rsr/` უნდა ვიყუროთ.
> - root-ის `src/` ცარიელია — ძველი `Main.java` წაშლილია, პრობლემა აღარაა.

### 7.2 შემდეგი ნაბიჯები, რიგითობით

1. ✅ **Neon-ის ბაზა** — მიერთებულია. კონფიგურაცია: `application.properties` (`spring.profiles.active=local`, `ddl-auto=validate`) + `application-local.properties` credentials-ით, gitignore-ში.
2. **დამოკიდებულებების დამატება** — `jjwt` (იხ. 7.3); `springdoc` სურვილისამებრ, თუ Boot 4-თან აეწყო.
3. ✅ **პირველი Flyway მიგრაცია** — `V1__init.sql` დაწერილია და **გაშვებულია**, იხ. §8. აქვე შევიდა `teams` / `team_members` და nullable `game_sessions.team_id`, თუმცა ეტაპი 1-ში ცარიელი რჩება.
4. **პირველი vertical slice** — entity → repository → service → controller `questions`-ისთვის, რომ ჯაჭვი ბაზიდან HTTP-მდე გაიტესტოს.
5. **git commit** — `backend/` ჯერ untracked-ია.

### 7.3 ხელით დასამატებელი დამოკიდებულებები — `build.gradle` (Groovy სინტაქსი)

```groovy
    // JWT — მხოლოდ api-ა კოდში ხილული, დანარჩენი ორი runtime-ია.
    // Spring-ზე დამოკიდებული არაა → Boot 4-ზე უპრობლემოდ მუშაობს.
    implementation 'io.jsonwebtoken:jjwt-api:0.12.6'
    runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.12.6'
    runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.12.6'

    // Swagger UI — http://localhost:8080/swagger-ui.html
    // ⚠️ ვერსია Boot 4-თან თავსებადობაზე შესამოწმებელია (2.8.5 Boot 3-ისთვისაა).
    // implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:<ვერსია>'
```

> `flyway-database-postgresql` და `postgresql` დრაივერი Initializr-მა **უკვე დაამატა** — ხელით არ სჭირდება.

**კონფიგურაციის სიტყვები:**

| სიტყვა | როდის |
|---|---|
| `implementation` | კოდში `import`-ით იყენებ |
| `runtimeOnly` | მხოლოდ გაშვებისას სჭირდება (დრაივერები, `jjwt-impl`) |
| `testImplementation` | მხოლოდ ტესტებში |
| `compileOnly` + `annotationProcessor` | კოდის გენერატორები (Lombok) |

**რატომ აქვს ზოგს ვერსია და ზოგს არა:** `io.spring.dependency-management` პლაგინი Spring Boot-ის **BOM**-ს იყენებს — თავსებადობაზე შემოწმებული ვერსიების სიას. თუ ბიბლიოთეკა BOM-შია (Spring starter-ები, Flyway, Postgres დრაივერი, Jackson, Hibernate) — ვერსიას **არ წერ**. `springdoc` და `jjwt` მესამე მხარისაა და BOM-ში არ არიან → ვერსია ხელით.

ვერსიების შემოწმება: IntelliJ-ში ნომერზე `Ctrl+Space`, ან [central.sonatype.com](https://central.sonatype.com).

### 7.4 ცვლილების ჩატვირთვა

`build.gradle`-ის შენახვის შემდეგ IntelliJ ზემოთ მარჯვნივ გამოიტანს **„Load Gradle Changes"** (ან სპილოს ხატულა Gradle-ის პანელში, ან `Ctrl+Shift+O`). პირველი ჩამოტვირთვა 1-2 წუთია.

შემოწმება:

```bash
./gradlew dependencies --configuration runtimeClasspath | grep -i jjwt
```

### 7.5 Neon-თან დაკავშირება — გავლილი ხაფანგები

connection string libpq-ის ფორმატშია და JDBC-სთვის გარდაქმნას საჭიროებს:

| წესი | |
|---|---|
| პრეფიქსი | `postgresql://` → `jdbc:postgresql://` |
| user და password | URL-იდან ამოღებული, ცალკე property-ებში |
| `channel_binding=require` | **იშლება** — libpq-ის პარამეტრია, JDBC ვერ ცნობს |
| `sslmode=require` | **რჩება** — Neon SSL-ის გარეშე კავშირს არ დაუშვებს |
| host | **direct, `-pooler`-ის გარეშე** |

`-pooler`-ის უკან PgBouncer ზის transaction mode-ში, სადაც სესიის მდგომარეობა კავშირებს შორის არ ნარჩუნდება — Flyway კი მიგრაციისას session-level advisory lock-ს იღებს. HikariCP-ს ისედაც აქვს თავისი პული.

**ორი შეცდომა, რომელიც რეალურად დაგვემართა:**

1. `user:password` მთლიანად ჩაიწერა `username`-ში (ე.ი. პაროლი ორჯერ). სიმპტომი — `FATAL: password authentication failed`.
2. host-ის დაბოლოება გაორმაგდა (`...neon.tech.eu-central-1.aws.neon.tech`). სიმპტომი — `PSQLException: ERROR: Endpoint ID is not specified`. `*.neon.tech` DNS-ში wildcard-ია, ამიტომ კავშირი proxy-მდე მიდის, მაგრამ SNI-ში წასულ სახელში Neon endpoint ID-ს ვერ პოულობს.

დრაივერი **pgjdbc 42.7.11**-ია და SNI-ს სრულად უჭერს მხარს — Neon-ის დოკუმენტაციაში ნახსენები `options=endpoint%3D...` შემოვლითი გზა **არ გვჭირდება**.

---

## 8. მონაცემთა სქემა

**რეალიზებულია და გაშვებულია:** `backend/src/main/resources/db/migration/V1__init.sql` (11 ცხრილი, 12 ინდექსი). ჭეშმარიტების წყარო ის ფაილია — ქვემოთ მოცემული ჩონჩხი მხოლოდ ორიენტირია.

> ⚠️ **V1 გაყინულია.** მიგრაცია Neon-ზე შესრულდა 2026-08-20-ს (`Successfully applied 1 migration to schema "public", now at version v1`). Flyway-მ მისი checksum ბაზაში ჩაწერა — ფაილის ერთი სიმბოლოს შეცვლაც კი შემდეგ გაშვებას `Migration checksum mismatch`-ით ჩააგდებს. **სქემის ნებისმიერი ცვლილება ამიერიდან `V2__*.sql`-ია.**

### 8.1 მიღებული გადაწყვეტილებები (2026-08-20)

- **UUID პირველადი გასაღებები**, `gen_random_uuid()`-ით (PostgreSQL 13+, extension არ სჭირდება). `BIGSERIAL` უარყოფილია: თანმიმდევრული ID საჯარო URL-ში კონტენტის მოცულობას ამხელს.
- **`VARCHAR` + `CHECK`, არა native `ENUM`.** მნიშვნელობის დამატება ერთხაზიანი `ALTER`-ია, წაშლა/გადარქმევა კი native enum-ში მტკივნეული; JPA-ც სტრიქონს დამატებითი კონფიგურაციის გარეშე ალაგებს.
- **`TIMESTAMPTZ` ყველგან.**
- **`ON DELETE` პოლიტიკა შეგნებულია და §4.4-ის წესებს აღასრულებს:**
  - `questions.author_id` → **RESTRICT** — ავტორის კრედიტი გამოქვეყნებულ კითხვას ფეხქვეშ არ უნდა გამოეცალოს.
  - `package_questions.question_id` → **RESTRICT** — სხვის პაკეტში მოხვედრილი კითხვა ვერ წაიშლება; გზა არის `ARCHIVED`.
  - `session_answers.question_id` → **SET NULL** (სვეტი nullable-ია) — snapshot-ები ისედაც ინახავს ყველაფერს საჩვენებლად, ე.ი. ისტორია ხელუხლებელი რჩება.
- **`users.is_blocked`** პირველივე მიგრაციაშია — §6-ის მაღაზიების მოთხოვნა.
- ძებნის ინდექსი: `to_tsvector('simple', text)` GIN-ით. `'simple'` სწორია — PostgreSQL-ს ქართული ლექსიკონი არ მოყვება.

### 8.2 ცხრილების ჩონჩხი

```
users              (id, email, display_name, role, created_at)
questions          (id, author_id, text, answer, comment, source,
                    handout_text, handout_media_url,
                    visibility, status, created_at, updated_at)
question_answers   (id, question_id, accepted_text)          -- ალტერნატიული პასუხები
question_parts     (id, question_id, position, text, answer)  -- DUPLET / BLITZ
packages           (id, author_id, title, theme, visibility, status, share_code)
package_questions  (package_id, question_id, position, tour_number)
game_sessions      (id, user_id, package_id, team_id NULL, started_at, finished_at)
session_answers    (id, session_id, question_id,
                    question_text_snapshot, answer_snapshot,
                    user_answer, is_correct, answered_at)
teams              (id, name, owner_id)                       -- ეტაპი 2, ცარიელი
team_members       (team_id, user_id, role)                   -- ეტაპი 2, ცარიელი
reports            (id, reporter_id, question_id, reason, status, created_at)
```

---

## 9. ღია საკითხები

- საჯარო კატალოგის მოდერაციის დატვირთვა solo-დეველოპერისთვის — გადაწყდა „თავიდანვე საჯარო", მაგრამ პრაქტიკაში შეიძლება მოგვიწიოს რიგის შეზღუდვა ან ავტო-დამტკიცების წესები.
- მონეტიზაციის მოდელი — არ განგვიხილავს.
- ~~Neon-ის ანგარიში~~ — დახურულია, ბაზა მუშაობს (2026-08-20).
- `springdoc-openapi`-ს Boot 4-თან თავსებადობა შეუმოწმებელია (იხ. §2.1).
- Neon-ის უფასო ტარიფზე compute ~5 წუთის უმოქმედობის შემდეგ იძინებს — პირველი მოთხოვნა cold start-ის შემდეგ რამდენიმე წამია. HikariCP-ის timeout-ები ამის გათვალისწინებითაა (`connection-timeout=30000`).
- `rootProject.name = 'backend'` vs `rsr` — გადარქმევა გადასაწყვეტია, სანამ კოდი დაიწერება.
- git რეპოზიტორი ინიციალიზებულია (`Initial commit`), მაგრამ `backend/` ჯერ commit-ში არ არის. ეს საქაღალდე პერსონალურ ანგარიშზეა გამიზნული — იხ. `Desktop/Personal/.gitconfig_personal`.

---

## 10. სამუშაო წესები

- **არაფერი შეიქმნას ან შეიცვალოს პროექტში ნებართვის გარეშე** — scaffolding-ს და დამოკიდებულებების დამატებას მომხმარებელი თვითონ აკეთებს IntelliJ-იდან.
- ახსნა ჯერ, კოდი მერე.
- საუბრის ენა — ქართული.
