# MarketPulse

Self-hosted market data platform rozwijana jako projekt edukacyjny i portfolio:
Python, React, systemy rozproszone oraz infrastruktura Kubernetes na Raspberry Pi 5.
System ma pobierać dane rynkowe, przechowywać historię, wyświetlać wykresy i obliczać
statystyki oraz alerty. Zakres obejmuje analizę danych, bez realizacji transakcji.

## Status

Repozytorium zawiera na razie strukturę i dokumentację projektu. Aplikacje, obrazy
kontenerów, manifesty i pipeline CI/CD nie zostały jeszcze zaimplementowane.
Nie ma jeszcze polecenia uruchamiającego całą platformę.

## Stack i środowisko docelowe

| Obszar | Technologia / przeznaczenie |
| --- | --- |
| Frontend | Next.js, React, TypeScript — dashboard, wykresy, watchlisty |
| API | Python, FastAPI — HTTP API dla frontendu |
| Przetwarzanie | Python — collector, analytics, alerts |
| Dane | PostgreSQL — notowania, watchlisty, wyniki analiz, reguły alertów |
| Zdarzenia | NATS, docelowo JetStream dla trwałego przetwarzania |
| Runtime | Kontenery Linux ARM64, K3s na Raspberry Pi 5 z SSD 1 TB |
| Deployment | Helm, Argo CD, GitOps |
| CI i obrazy | Docelowo GitHub Actions i GHCR |
| Obserwowalność | Prometheus, Grafana, Loki, OpenTelemetry |

Docelowy host to `rpi5-01`, z Ubuntu Server ARM64. Konfigurację klastra oraz
parametry zasobów będziemy potwierdzać przy wdrażaniu. Laptop służy do developmentu.

## Struktura monorepo

```text
marketpulse/
├── apps/
│   ├── frontend/        # Next.js
│   └── api/             # FastAPI
├── services/
│   ├── collector/       # Pobieranie i normalizacja danych
│   ├── analytics/       # Asynchroniczne obliczenia
│   └── alerts/          # Reguły i zdarzenia alertów
├── infra/
│   ├── k8s/             # Bootstrap klastra i zasoby poza chartami aplikacji
│   ├── helm/            # Charty i konfiguracja wdrożeń
│   └── argocd/          # Definicje aplikacji GitOps
├── docs/
│   └── architecture.md
├── AGENTS.md
├── README.md
└── .gitignore
```

Puste katalogi zachowują w Git pliki `.gitkeep`; usuń je przy dodawaniu zawartości.

## Roadmapa

Wszystkie etapy poniżej są planowane.

1. **v1 — pierwszy przepływ danych:** AAPL/MSFT/NVDA → collector → PostgreSQL
   → FastAPI → wykres Next.js. Najpierw jeden dostawca i jeden interwał.
2. **v2 — watchlisty i porównania:** własne listy, porównywanie instrumentów,
   wykresy normalizowane do 100 i zmiany 1D/1W/1M/YTD.
3. **v3 — przetwarzanie asynchroniczne:** NATS, analytics worker, stopy zwrotu,
   średnie kroczące, zmienność, korelacje i drawdown.
4. **v4 — alerty cenowe:** reguły progowe, historia wywołań, deduplikacja.
5. **v5 — aktualizacje live:** WebSocket; częstotliwość zależna od dostawcy danych.
6. **v6 — obserwowalność:** Prometheus/Grafana, Loki i OpenTelemetry.
7. **v7 — pełny GitOps i CI/CD:** GitHub Actions → GHCR → Helm → Argo CD.
8. **v8 — odporność:** testy restartów podów, ponownego przetwarzania zdarzeń
   oraz odtwarzania danych z backupu.

Infrastrukturę dokładamy stopniowo: przygotowanie hosta i SSD, K3s, pierwszy
deployment, PostgreSQL z trwałym storage, ingress i Helm. Podstawowe logi,
health checks i obsługa błędów powstają razem z aplikacjami.

## Pierwszy etap implementacji

- Wybrać interwał i zweryfikować dostawcę danych; Twelve Data jest kandydatem
  z wcześniejszych ustaleń, a limity i warunki użycia wymagają sprawdzenia.
- Zdefiniować model instrumentu i świecy OHLCV oraz migracje PostgreSQL.
- Zaimplementować idempotentny import i odczyt historii przez FastAPI.
- Dodać pojedynczy wykres w Next.js, następnie kontenery i wdrożenie na K3s.

Publiczne demo powinno używać danych, które można legalnie udostępniać;
alternatywą są dane syntetyczne. Sekrety i lokalne dane pozostają poza repozytorium.

Szczegóły przepływu danych i decyzje projektowe: [architektura](docs/architecture.md).
Zasady dalszej pracy: [AGENTS.md](AGENTS.md).
