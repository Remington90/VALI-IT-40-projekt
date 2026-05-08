# Investing Journal
**Full Project Plan & Architecture**
*Spring Boot + Vue.js + PostgreSQL + Alpha Vantage API*

Java Class Group Project | 2-Week Sprint | 3 Developers

---

# 1. Project Overview

The Investing Journal is a full-stack web application that allows users to track their stock investments, monitor portfolio performance, and maintain personal trade notes. It follows a RESTful CRUD architecture with a Spring Boot backend and Vue.js frontend.

| Item | Choice |
|------|--------|
| Backend | Spring Boot (Java) |
| Frontend | Vue.js 3 + Vite |
| Database | PostgreSQL |
| Auth | Spring Security + JWT |
| Stock Data API | Alpha Vantage |
| ORM | Spring Data JPA / Hibernate |
| API Style | REST (JSON) |
| Version Control | Git (GitHub) |

---

# 2. Feature Scope

## Week 1 — Core (Must Finish)

| Feature | Backend | Frontend | Owner |
|---------|---------|----------|-------|
| User Register / Login | Spring Security + JWT | Login & Register pages | Person A |
| Trade Log CRUD | /api/trades endpoints | Add / Edit / Delete form | Person B |
| Portfolio Overview | P&L calculation logic | Dashboard with totals | Person B |
| Watchlist CRUD | /api/watchlist endpoints | Add / Remove tickers | Person C |
| Live Ticker Price | Alpha Vantage integration | Price on watchlist | Person C |

## Week 2 — Extras (Finish What You Can)

| Feature | Complexity | Owner Suggestion |
|---------|------------|-----------------|
| Trade notes / thesis field | Very Low | Person B |
| Average cost basis calculator | Low | Person B |
| Realised vs unrealised P&L | Low–Medium | Person B |
| CSV export | Low | Person A |
| Price alerts | Medium | Person A |
| Dividend tracker | Low | Person B |
| Sector breakdown | Medium | Person C |
| Historical portfolio chart | High | Person C |
| Benchmark comparison (vs S&P 500) | High | Person C |
| Multiple portfolios per user | Medium | Person A / B |

---

# 3. Architecture

## High-Level Overview

```
Vue.js (frontend)  ←→  Spring Boot REST API  ←→  PostgreSQL
                              ↕
                   Alpha Vantage API (external)
```

The frontend communicates with the backend exclusively via HTTP/REST (JSON). The backend handles all business logic, auth, and external API calls. The frontend never talks to Alpha Vantage directly — all ticker requests go through the backend, which caches results in the database.

## Spring Boot Package Structure

```
src/main/java/com/investingjournal/
├── auth/
│   ├── AuthController.java       POST /api/auth/register, /api/auth/login
│   ├── AuthService.java
│   ├── JwtUtil.java
│   └── JwtFilter.java
├── user/
│   ├── UserController.java       GET/PUT/DELETE /api/users/{id}
│   ├── UserService.java
│   ├── UserRepository.java
│   └── User.java                 Entity
├── trade/
│   ├── TradeController.java      GET/POST/PUT/DELETE /api/trades
│   ├── TradeService.java
│   ├── TradeRepository.java
│   └── Trade.java                Entity
├── portfolio/
│   ├── PortfolioController.java  GET /api/portfolio
│   └── PortfolioService.java     P&L calculation logic
├── watchlist/
│   ├── WatchlistController.java  GET/POST/DELETE /api/watchlist
│   ├── WatchlistService.java
│   ├── WatchlistRepository.java
│   └── WatchlistItem.java        Entity
├── ticker/
│   ├── AlphaVantageService.java  External API calls
│   ├── TickerCacheRepository.java
│   └── TickerCache.java          Entity (caches prices)
├── alert/
│   ├── AlertController.java      GET/POST/PUT/DELETE /api/alerts
│   ├── AlertService.java
│   └── Alert.java                Entity
└── config/
    ├── SecurityConfig.java
    └── WebConfig.java            CORS settings
```

## Vue.js Frontend Structure

```
src/
├── views/
│   ├── LoginView.vue
│   ├── RegisterView.vue
│   ├── DashboardView.vue         Portfolio overview
│   ├── TradesView.vue            Trade log
│   ├── WatchlistView.vue
│   └── AlertsView.vue
├── components/
│   ├── TradeForm.vue             Add / Edit trade
│   ├── PortfolioSummary.vue
│   ├── TickerSearch.vue          Symbol search input
│   └── PriceCard.vue
├── services/
│   ├── api.js                    Axios base config
│   ├── authService.js
│   ├── tradeService.js
│   └── watchlistService.js
├── stores/
│   └── authStore.js              Pinia — JWT token storage
└── router/
    └── index.js                  Vue Router — route guards
```

---

# 4. Data Model

| Entity | Key Fields |
|--------|-----------|
| User | id, username, email, passwordHash, createdAt |
| Trade | id, userId, ticker, type (BUY/SELL), price, quantity, fees, date, notes, sector |
| WatchlistItem | id, userId, ticker, targetPrice, notes, addedAt |
| TickerCache | id, symbol, price, changePercent, volume, lastFetched |
| Alert | id, userId, ticker, targetPrice, direction (ABOVE/BELOW), triggered, createdAt |
| Dividend | id, userId, ticker, amount, payDate |

## Relationships

- One User → many Trades
- One User → many WatchlistItems
- One User → many Alerts
- One User → many Dividends
- TickerCache is shared across all users (no foreign key to User)

---

# 5. REST API Endpoints

## Auth (Person A)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /api/auth/register | Create new user account |
| POST | /api/auth/login | Login, returns JWT token |

## Trades (Person B)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /api/trades | Get all trades for current user |
| GET | /api/trades/{id} | Get single trade |
| POST | /api/trades | Create new trade entry |
| PUT | /api/trades/{id} | Update trade entry |
| DELETE | /api/trades/{id} | Delete trade entry |
| GET | /api/portfolio | Get portfolio summary (P&L, totals) |

## Watchlist (Person C)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /api/watchlist | Get user's watchlist |
| POST | /api/watchlist | Add ticker to watchlist |
| PUT | /api/watchlist/{id} | Update notes or target price |
| DELETE | /api/watchlist/{id} | Remove ticker from watchlist |
| GET | /api/ticker/{symbol}/price | Fetch live price (cached) |
| GET | /api/ticker/search?q= | Search tickers by name |

## Alerts (Person A — Week 2)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /api/alerts | Get all alerts for user |
| POST | /api/alerts | Create price alert |
| PUT | /api/alerts/{id} | Update alert target |
| DELETE | /api/alerts/{id} | Delete alert |

---

# 6. Alpha Vantage Integration

## Free Tier Limit

25 API requests per day. This is the most important constraint to design around.

## Endpoints You Will Use

| Data Needed | Alpha Vantage Function | Used For |
|-------------|----------------------|----------|
| Current price + change % | GLOBAL_QUOTE | Watchlist, portfolio P&L |
| Search ticker by name | SYMBOL_SEARCH | Ticker search input |
| Company info | OVERVIEW | Optional detail page |
| Historical prices | TIME_SERIES_DAILY | Portfolio chart (Week 2) |

## Caching Strategy

Before every Alpha Vantage call, check TickerCache. If `lastFetched` is within 15 minutes, return the cached value. Only call the external API if the cache is stale. This keeps you well within the 25/day limit.

## Service Sketch

```java
@Service
public class AlphaVantageService {

    @Value("${alphavantage.api.key}")
    private String apiKey;

    public GlobalQuoteDto getQuote(String symbol) {
        // 1. Check TickerCache — return if fresh (< 15 min)
        // 2. If stale, call Alpha Vantage GLOBAL_QUOTE
        // 3. Save result to TickerCache
        // 4. Return data
    }
}
```

Store your API key in `application.properties` and add that file to `.gitignore` — never commit API keys to GitHub.

---

# 7. Authentication (JWT)

Because the frontend (Vue.js) and backend (Spring Boot) are separate, you use JWT (JSON Web Tokens) rather than session-based auth. This is the industry standard for this setup.

## Flow

1. User submits login form → Vue calls `POST /api/auth/login`
2. Spring Boot validates credentials, returns a signed JWT token
3. Vue stores the token in Pinia (memory) or localStorage
4. Every subsequent API request includes the token in the `Authorization` header
5. Spring Boot's `JwtFilter` validates the token on every request

## Dependencies to add to pom.xml

- `spring-boot-starter-security`
- `jjwt-api` (io.jsonwebtoken)
- `jjwt-impl`
- `jjwt-jackson`

---

# 8. Team Split & Timeline

## Person A — Auth + User + Alerts

- **Week 1:** Spring Security setup, JWT, register/login endpoints, login/register Vue pages
- **Week 2:** Price alerts CRUD, CSV export endpoint

## Person B — Trades + Portfolio

- **Week 1:** Trade entity + full CRUD endpoints, P&L calculation service, portfolio summary endpoint, trade log Vue page
- **Week 2:** Cost basis, realised/unrealised P&L, trade notes field, dividend tracker

## Person C — Ticker API + Watchlist

- **Week 1:** Alpha Vantage service + caching, watchlist CRUD endpoints, watchlist Vue page with live prices
- **Week 2:** Ticker search input, sector breakdown, historical chart (if time)

## Suggested Day-by-Day

| Days | Goal |
|------|------|
| 1–2 | Set up Spring Boot project, PostgreSQL, GitHub repo with shared structure. Everyone aligns on entity classes and packages. |
| 3–4 | Person A: working auth endpoints. Person B: Trade CRUD working in Postman. Person C: Alpha Vantage returning prices. |
| 5–7 | All Week 1 backend endpoints working. Start Vue frontend — each person builds pages for their feature. |
| 8–9 | Full Week 1 feature working end-to-end (Vue ↔ Spring ↔ DB). Fix bugs. This is the milestone. |
| 10–12 | Add Week 2 features in priority order. Don't start new features if Week 1 has bugs. |
| 13–14 | Polish, testing, fix edge cases, prepare demo. |

---

# 9. Git Workflow

Use feature branches — never commit directly to main.

- `main` — always working, demo-ready code
- `dev` — integration branch, merge features here first
- `feature/auth` — Person A's branch
- `feature/trades` — Person B's branch
- `feature/watchlist` — Person C's branch

Pull Request to `dev` when a feature is done → one teammate reviews → merge. Merge `dev` to `main` only when everything works.

## Files to add to .gitignore

```
application.properties   (contains API key and DB password)
*.env
node_modules/
.idea/
target/
```

---

# 10. Suggested Libraries & Dependencies

## Backend (pom.xml)

| Dependency | Purpose |
|------------|---------|
| spring-boot-starter-web | REST API |
| spring-boot-starter-data-jpa | Database ORM |
| spring-boot-starter-security | Auth |
| postgresql | DB driver |
| jjwt-api / jjwt-impl | JWT tokens |
| lombok | Reduce boilerplate (@Getter, @Setter etc.) |
| spring-boot-starter-validation | Input validation (@NotNull, @Email etc.) |

## Frontend (npm)

| Package | Purpose |
|---------|---------|
| vue@3 + vite | Core framework + build tool |
| vue-router | Page routing + auth guards |
| pinia | State management (JWT token, user) |
| axios | HTTP requests to backend |
| chart.js + vue-chartjs | Charts (portfolio history, sector breakdown) |