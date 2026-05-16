# LLM jako most między badaniami jakościowymi a ilościowymi w psychologii i socjologii

## 1. Problem

Badania jakościowe — wywiady pogłębione, fokusy, etnografia, dzienniki — dostarczają bogatego materiału, ale są drogie w obróbce i trudne do skorelowania z danymi ilościowymi (kwestionariusze, dane administracyjne, panele). Klasyczna ścieżka prowadzi przez kodowanie ręczne w NVivo / Atlas.ti / MAXQDA, agregację do wskaźników i dopiero potem korelację z liczbami. Wąskim gardłem jest czas oraz wariancja między kodującymi — Krippendorffa α często ledwo przekracza 0.7.

LLM zmienia to równanie po dwóch stronach: kodowanie staje się 100× tańsze i 1000× szybsze, ale pojawia się nowy problem — model jako "koder" trzeba walidować, bo niesie własne uprzedzenia teoretyczne i kulturowe wynikające z korpusu treningowego.

## 2. LLM jako klasyfikator danych jakościowych

Najprostsze zastosowanie: *codebook → prompt*. Klasyczny podręcznik kodowania (kategorie, definicje, przykłady pozytywne i negatywne) staje się systemowym promptem. Każdy fragment transkryptu wraca z modelu jako ustrukturyzowany JSON:

```json
{
  "fragment_id": "P07-min12-15",
  "categories": ["wsparcie_emocjonalne", "izolacja_społeczna"],
  "intensity": 0.7,
  "confidence": 0.85,
  "evidence_quote": "..."
}
```

Trzy techniki podnoszą wiarygodność:
- **Wielu sędziów** — ten sam fragment kodowany przez Claude, GPT-4 i Gemini; zgodność = wskaźnik pewności.
- **Walidacja na próbce** — 10–15% materiału kodowane równolegle przez człowieka i LLM, raportujemy Krippendorffa α dla pary człowiek–model.
- **Active learning** — fragmenty o niskim konsensusie idą do człowieka; reszta wyłącznie do LLM.

## 3. LLM jako transformator: qual → quant

Klasyfikacja kategoryczna to dopiero początek. Mocniejsza warstwa: **przekład tekstu swobodnego na skale ilościowe**. Przykład: pacjent w wywiadzie mówi 200 słów o swoim samopoczuciu. LLM dostaje prompt z definicją skali (PHQ-9, BDI-II) i zwraca przewidywany score per item — wraz z uzasadnieniem opartym o konkretne cytaty. Wynik: macierz `respondent × pytanie skali`, którą można porównać 1:1 z kwestionariuszem wypełnionym przez tego samego respondenta.

Otwiera to trzy ścieżki:
- **Cross-method validation** — czy LLM wyciąga z wywiadu te same score'y, które respondent zadeklarował w ankiecie?
- **Latent constructs** — wyodrębnianie konstruktów ukrytych ("poczucie sprawczości", "kapitał kulturowy") tam, gdzie respondent ich nie nazywa wprost.
- **Vector embeddings** — każdy wywiad jako wektor w przestrzeni 1536-wymiarowej; klasteryzacja daje typologię jakościową skorelowaną z grupami demograficznymi.

## 4. Korelacja z danymi ilościowymi — trzy poziomy

**Poziom A — Ten sam respondent (within-subject).** Wywiad i kwestionariusz tego samego uczestnika. LLM transformuje wywiad do tej samej skali. Korelacja Spearmana ρ między obiema wersjami mówi, czy człowiek **mówi** to samo, co **deklaruje**. Rozjazd ujawnia social desirability bias albo problemy semantyczne skali.

**Poziom B — Agregacja populacyjna.** Kategorie z LLM łączone z danymi ESS, EVS, Diagnozy Społecznej. Pytanie typu: czy częstotliwość kategorii "lęk ekonomiczny" w wywiadach koreluje z regionalnym wskaźnikiem bezrobocia? Klasyczna triangulacja, tylko że jakościowa strona jest teraz skalowalnie kodowana.

**Poziom C — Predykcja krzyżowa.** Model przewidujący wynik jakościowy z ilościowego (i odwrotnie). Błąd predykcji = miara "niezależnej informacji" jakościowej — tego, czego ankieta nie wychwyciła.

## 5. Pipeline

```
Audio wywiadu
   ↓ Whisper (lokalnie)
Transkrypt
   ↓ pseudonimizacja (PII → tokeny)
Pseudonimowany tekst
   ↓ LLM 1: klasyfikator kategorii
   ↓ LLM 2: ekstraktor skal
JSON {respondent_id, kategorie[], scale_predictions{}}
   ↓ walidacja: 10% manualne kodowanie
Macierz danych (CSV / parquet)
   ↓ R / Python / Stata
Korelacje, regresje, SEM
```

## 6. Ograniczenia

- **Stronniczość LLM** jest realna. Model wytrenowany głównie na anglojęzycznym korpusie inaczej rozumie "godność" niż polski etnograf. Cross-cultural research wymaga oddzielnej walidacji per język.
- **Konstrukty powstają w prompcie.** To, co model "znajduje", jest funkcją definicji, którą mu daliśmy. Grounded theory w czystej postaci wymaga człowieka.
- **Niedeterminizm.** Ten sam fragment dwukrotnie zakodowany może dać różne wyniki. Konieczne: `temperature = 0` plus wielokrotne uruchomienia z raportowaniem zgodności.
- **Etyka i prywatność.** Wywiady zawierają dane wrażliwe. Pseudonimizacja przed wysyłką do chmurowego LLM (albo lokalne modele klasy `mistral-nemo`) jest niezbędna; zgoda IRB obejmująca przetwarzanie przez AI również.
- **LLM nie zastępuje teorii.** Klasyfikator jest tak dobry jak codebook; codebook jest tak dobry jak teoria, która go zrodziła.

## 7. Trzy konkretne zastosowania

1. **Satysfakcja z opieki zdrowotnej** — wywiady z pacjentami + SF-36. LLM ekstraktuje predykcje SF-36 z wywiadu, korelujemy z faktycznie wypełnionym kwestionariuszem. Rozjazd ujawnia obszary, których standardowa skala nie pokrywa.
2. **Kapitał społeczny w Polsce** — narracje z 200 wywiadów kodowane na wymiary Putnama (bonding / bridging / linking), korelowane z Diagnozą Społeczną na poziomie województw.
3. **Cross-cultural psychology** — ten sam codebook stosowany do wywiadów w PL / EN / DE / JP, z jawnym pomiarem wariancji LLM między językami jako proxy dla *measurement invariance*.

---

**Wniosek.** LLM nie zastępuje badacza jakościowego — przesuwa wąskie gardło z kodowania na walidację. Pole mixed-methods staje się skalowalne pod warunkiem, że badacz traktuje LLM tak samo poważnie jak każdy inny instrument pomiarowy: z walidacją na próbie kontrolnej, jawną dokumentacją wariancji i raportowaniem stronniczości modelu.
