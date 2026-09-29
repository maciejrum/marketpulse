# MarketPulse — kontekst i zasady pracy

## Cel i aktualny stan

Budujemy self-hosted market data platform do nauki infrastruktury i jako projekt
portfolio. Docelowe środowisko: Raspberry Pi 5, SSD 1 TB, Linux ARM64 i K3s.
Priorytetem jest zrozumiały przepływ danych i stopniowe poznawanie infrastruktury.
Nie implementujemy wykonywania zleceń giełdowych.

Repo zawiera obecnie tylko szkielet i dokumentację. Nie zakładaj istnienia
działających serwisów, testów, klastra ani pipeline. Czytaj README.md i
docs/architecture.md przed zmianami; aktualizuj status wraz z implementacją.

## Ustalony stack i granice

- `apps/frontend`: Next.js, React, TypeScript; komunikacja z HTTP API.
- `apps/api`: Python/FastAPI; moduły market, watchlists i alert rules w jednym API
  na start. Nie rozdzielaj ich przedwcześnie na dodatkowe mikroserwisy.
- `services/collector`: Python; dostawca danych, normalizacja i zapis notowań.
- `services/analytics`: Python worker; obliczenia na danych rynkowych.
- `services/alerts`: Python worker; ocena reguł i zapis wywołanych alertów.
- PostgreSQL jako trwały magazyn, NATS jako broker zdarzeń.
- K3s, Helm i Argo CD; docelowo GitHub Actions oraz GHCR.
- Prometheus/Grafana dla metryk, Loki dla logów, OpenTelemetry dla instrumentacji.

## Sposób realizacji

1. Zacznij od v1: AAPL/MSFT/NVDA → collector → PostgreSQL → API → wykres.
2. Realizuj kolejne etapy roadmapy z README; nie instaluj całego stacku z góry.
3. Traktuj katalogi jako granice odpowiedzialności i przyszłych obrazów kontenerów.
4. Wybieraj rozwiązania mieszczące się na jednym Pi. Nie zakładaj HA, wielu węzłów
   ani konkretnej ilości RAM bez sprawdzenia środowiska.
5. Dokumentuj nowe decyzje i odróżniaj propozycje od wdrożonej funkcjonalności.
6. Dokumentację pisz po polsku, nazwy w kodzie i kontrakty API po angielsku.

## Dane i niezawodność

- Oddziel kod dostawcy od modelu domenowego; limity API muszą być konfigurowalne.
- Stosuj timeouty, ograniczone retry z backoffem i obsługę rate limitów.
- Używaj UTC dla timestampów, jawnej waluty i precyzyjnych typów dla cen.
- Importy i konsumenci zdarzeń muszą tolerować powtórzenia. Nie obiecuj
  przetwarzania exactly-once; projektuj deduplikację i bezpieczne ponowienia.
- Zmiany schematu bazy prowadź przez wersjonowane migracje.
- Nie dodawaj zależności od zewnętrznego API do testów jednostkowych; używaj fixtures.
- Nie kopiuj do publicznego repo ani demo danych bez sprawdzenia praw do ich użycia.

## Wdrożenia i sekrety

- Zapewnij obrazy `linux/arm64`; dodatkowe architektury dodawaj w miarę potrzeby.
- Dobieraj wersje zależności przy implementacji, commituj lockfile i pinuj obrazy.
- Manifesty aplikacji utrzymuj w Helm. `infra/k8s` służy do bootstrapu i zasobów
  poza chartami; nie twórz dwóch źródeł konfiguracji tego samego zasobu.
- Dodawaj probes odpowiednie dla typu procesu oraz requests/limits przy wdrażaniu.
- PostgreSQL i trwały broker wymagają PVC, retencji i planu backupu/restore.
- Nie commituj `.env`, tokenów, kubeconfigów, kluczy ani jawnych Kubernetes Secrets.
  Pliki `.env.example` mogą zawierać wyłącznie atrapy. SOPS lub Sealed Secrets
  wybierz i udokumentuj przed wprowadzeniem sekretów do GitOps.
- Frontend nie łączy się bezpośrednio z bazą ani brokerem. Baza i NATS pozostają
  wewnętrzne; sposób publikacji aplikacji jest osobnym etapem.

## Weryfikacja

Uruchamiaj lint, sprawdzanie typów i testy właściwe dla zmienionego komponentu,
gdy zostaną dodane jego narzędzia. Przy zmianach w Helm sprawdzaj lint i renderowanie
chartów; przy obrazach weryfikuj ARM64. Dla samej dokumentacji sprawdź linki,
spójność nazw i `git diff --check`. Nie twórz pozornych testów dla pustego szkieletu.
W podsumowaniu podaj wykonane kontrole i ograniczenia; nie deklaruj wdrożenia
ani testów, których nie wykonano.
