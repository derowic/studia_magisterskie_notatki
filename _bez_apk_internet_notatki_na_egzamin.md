# Bezpieczeństwo Aplikacji Internetowych – Pytania Egzaminacyjne

**Notatki / zestaw pytań z odpowiedziami**

---

## 1. Czy możliwe jest stworzenie całkowicie bezpiecznej aplikacji internetowej?

**Nie.**  
Bezpieczeństwo to **proces**, a nie stan stały. Zawsze istnieje ryzyko wystąpienia podatności typu „zero-day”, błędów ludzkich lub nowych metod ataku. Można jedynie minimalizować ryzyko, ale nigdy całkowicie go wyeliminować.

---

## 2. Które podatności aplikacji internetowych stanowią obecnie największe zagrożenie dla użytkowników?

Według rankingu **OWASP Top 10** największe zagrożenia to:

- **Broken Access Control** – naruszenie kontroli dostępu
- **Cryptographic Failures** – błędy kryptograficzne (słabe szyfrowanie)
- **Injection** – wstrzyknięcia (SQL Injection, XSS itp.)
- **Insecure Design** – niestaranne projektowanie bezpieczeństwa od początku

---

## 3. Dlaczego walidacja danych wejściowych jest jednym z najważniejszych elementów bezpieczeństwa aplikacji?

Aplikacja nie powinna ufać żadnym danym pochodzącym od użytkownika.  
Brak walidacji umożliwia ataki takie jak:
- SQL Injection
- Cross-Site Scripting (XSS)
- Denial of Service (DoS)

Walidacja + sanityzacja to pierwsza linia obrony.

---

## 4. W jaki sposób ataki SQL Injection mogą prowadzić do przejęcia kontroli nad systemem?

Atakujący wstrzykuje złośliwy kod SQL do zapytania. Może to prowadzić do:
- Wykradzenia haseł administratorów
- Modyfikacji lub usunięcia całej bazy danych
- Wykonywania poleceń systemowych na serwerze (np. poprzez `xp_cmdshell`)

---

## 5. W jaki sposób organizacje mogą zwiększyć poziom bezpieczeństwa swoich aplikacji internetowych?

- Wdrażanie **Secure SDLC** (bezpieczny cykl życia oprogramowania)
- Regularne **testy penetracyjne** i audyty kodu
- Szkolenia programistów z **Secure Coding**
- Automatyczne skanowanie podatności (**SAST/DAST**)
- Używanie **Web Application Firewall (WAF)**

---

## 6. Jakie konsekwencje może mieć atak Cross Site Scripting (XSS) dla użytkowników?

- Kradzież ciasteczek sesyjnych (przejęcie konta)
- Wyłudzanie danych (phishing na zaufanej stronie)
- Przekierowanie na złośliwe strony
- Wykonywanie operacji w imieniu użytkownika

---

## 7. Dlaczego mechanizmy zarządzania sesją są kluczowe dla bezpieczeństwa aplikacji?

HTTP jest protokołem bezstanowym. Sesja pozwala „pamiętać”, że użytkownik jest zalogowany.  
Słabe zarządzanie sesją umożliwia **Session Hijacking** lub **Session Fixation**.

---

## 8. Jakie znaczenie mają aktualizacje oprogramowania w kontekście bezpieczeństwa?

Aktualizacje (patche) naprawiają znane luki bezpieczeństwa.  
Korzystanie z nieaktualnego oprogramowania to jedno z najczęstszych powodów udanych ataków.

---

## 9. Jakie narzędzia mogą być wykorzystywane do testowania bezpieczeństwa aplikacji internetowych?

- **Burp Suite**, **OWASP ZAP** (proxy)
- **Nessus**, **Nmap** (skanery)
- **SonarQube**, **Snyk** (analiza kodu)
- **Hashcat**, **John the Ripper** (łamanie haseł)

---

## 10. Jaką rolę odgrywają programiści w zapewnianiu bezpieczeństwa?

Programiści są **pierwszą linią obrony**.  
Ich zadaniem jest pisanie bezpiecznego kodu, stosowanie dobrych praktyk i rozumienie mechanizmów bezpieczeństwa.

---

## 11. Jak brak walidacji danych wejściowych prowadzi do SQL Injection? Jak się przed tym bronić?

Brak walidacji pozwala na konkatenację stringów z kodem SQL.  
**Najlepsza ochrona**: **Prepared Statements / Parametryzowane zapytania**.

---

## 12. Różnice pomiędzy XSS a CSRF + metody ochrony

| Atak   | Opis                                      | Ochrona                              |
|--------|-------------------------------------------|--------------------------------------|
| **XSS**    | Wstrzyknięcie skryptu JS                  | Escaping, **CSP**, HttpOnly cookies  |
| **CSRF**   | Wymuszenie niechcianego żądania           | **Anti-CSRF Tokeny**, SameSite cookies |

---

## 13. Na czym polega mechanizm zarządzania sesją i jakie są jego główne zagrożenia?

- Generowanie unikalnego **Session ID** po zalogowaniu.
- Zagrożenia: Session Hijacking, Session Fixation, brak wygasania sesji.

**Dobre praktyki**: długi losowy ID, flagi `Secure` + `HttpOnly`, krótkie timeouty.

---

## 14. Dlaczego testowanie bezpieczeństwa powinno być częścią procesu tworzenia oprogramowania?

Naprawa błędu bezpieczeństwa na produkcji jest **znacznie droższa** niż na etapie projektowania lub kodowania.

**Metody**: SAST, DAST, Pentesty, Code Review.

---

## 15. Jakie konsekwencje może mieć naruszenie bezpieczeństwa aplikacji dla organizacji?

- Finansowe (kary RODO, koszty naprawy)
- Reputacyjne (utrata zaufania klientów)
- Prawne (procesy sądowe)
- Operacyjne (przerwy w działaniu usług)

---

## 16. Na czym polega model CIA?

- **Confidentiality** (Poufność) – dane tylko dla uprawnionych
- **Integrity** (Integralność) – dane nie zostały zmienione
- **Availability** (Dostępność) – system jest dostępny gdy jest potrzebny

To podstawowy model bezpieczeństwa informatycznego.

---

## 17–24. (Pozostałe pytania – skrócone)

- **17.** Zarządzanie sesją – patrz pkt 7 i 13.
- **18.** **Authentication** (kim jesteś?) vs **Authorization** (co możesz robić?).
- **19.** SQL Injection narusza **Confidentiality** i **Integrity**.
- **20.** Aktualizacje zamykają znane luki.
- **21.** XSS – patrz pkt 6.
- **22.** Ochrona CSRF: tokeny + SameSite.
- **23.** Kluczowe nagłówki: **CSP**, **HSTS**, **X-Frame-Options**.
- **24.** **DevSecOps** = bezpieczeństwo w całym cyklu życia (od projektowania po produkcję).

---

**Powodzenia na egzaminie!**  
Jeśli chcesz wersję z większą ilością szczegółów lub przykładów kodu – daj znać.