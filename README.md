
#creazione tabella pulita
CREATE OR REPLACE TABLE `helpful-scion-492207-u1.Finance.Finance_cleaned` AS
SELECT 
    -- DEMOGRAFIA
    LOWER(TRIM(gender)) AS gender,
    CAST(age AS INT64) AS age,

    -- INVESTIMENTI (normalizzati)
    SAFE_CAST(Mutual_Funds AS FLOAT64) AS mutual_funds,
    SAFE_CAST(Equity_Market AS FLOAT64) AS equity_market,
    SAFE_CAST(Debentures AS FLOAT64) AS debentures,
    SAFE_CAST(Government_Bonds AS FLOAT64) AS government_bonds,
    SAFE_CAST(Fixed_Deposits AS FLOAT64) AS fixed_deposits,
    SAFE_CAST(PPF AS FLOAT64) AS ppf,
    SAFE_CAST(Gold AS FLOAT64) AS gold,

    -- COMPORTAMENTO
    LOWER(TRIM(Invest_Monitor)) AS invest_monitor,
    LOWER(TRIM(Expect)) AS expected_return,
    LOWER(TRIM(Duration)) AS investment_duration,

    -- MOTIVAZIONI
    LOWER(TRIM(Objective)) AS objective,
    LOWER(TRIM(Purpose)) AS purpose,
    LOWER(TRIM(Reason_Equity)) AS reason_equity,
    LOWER(TRIM(Reason_Mutual)) AS reason_mutual,
    LOWER(TRIM(Reason_Bonds)) AS reason_bonds,
    LOWER(TRIM(Reason_FD)) AS reason_fd,

    -- INFO SOURCE
    LOWER(TRIM(Source)) AS source

FROM `helpful-scion-492207-u1.Finance.Finance_cleaning`
WHERE age IS NOT NULL;

CREATE OR REPLACE TABLE `helpful-scion-492207-u1.Finance.Finance_enriched` AS
SELECT *,
    
    -- Risk score (chiave per analisi avanzata)
    (equity_market + gold) AS risk_score,

    -- Safe score
    (fixed_deposits + government_bonds + ppf) AS safe_score,

    -- Segmentazione età
    CASE 
        WHEN age < 25 THEN 'young'
        WHEN age BETWEEN 25 AND 35 THEN 'adult'
        ELSE 'senior'
    END AS age_group,

    -- Profilo rischio
    CASE 
        WHEN (equity_market + gold) > 10 THEN 'high_risk'
        WHEN (equity_market + gold) BETWEEN 5 AND 10 THEN 'medium_risk'
        ELSE 'low_risk'
    END AS risk_profile

FROM `helpful-scion-492207-u1.Finance.Finance_cleaned`;
SELECT
    gender,
    COUNT(*) AS total_users,
    AVG(age) AS avg_age,
    AVG(risk_score) AS avg_risk,
    AVG(safe_score) AS avg_safe
FROM `helpful-scion-492207-u1.Finance.Finance_enriched`
GROUP BY gender;

SELECT
    age_group,
    AVG(equity_market) AS equity,
    AVG(mutual_funds) AS mutual_funds,
    AVG(fixed_deposits) AS fixed_deposits
FROM `helpful-scion-492207-u1.Finance.Finance_enriched`
GROUP BY age_group
ORDER BY age_group;

#durata investimento vs rischio
SELECT
    investment_duration,
    AVG(risk_score) AS avg_risk,
    COUNT(*) AS users
FROM `helpful-scion-492207-u1.Finance.Finance_enriched`
GROUP BY investment_duration
ORDER BY avg_risk DESC;

#obiettivi vs comportamento

SELECT
    objective,
    AVG(risk_score) AS avg_risk,
    AVG(safe_score) AS avg_safe
FROM `helpful-scion-492207-u1.Finance.Finance_enriched`
GROUP BY objective
ORDER BY avg_risk DESC;

#fonte di informazione vs scelte
SELECT
    source,
    AVG(equity_market) AS equity,
    AVG(fixed_deposits) AS fd
FROM `helpful-scion-492207-u1.Finance.Finance_enriched`
GROUP BY source;

#distribuzione profili di rischio
SELECT
    risk_profile,
    COUNT(*) AS users,
    ROUND(100 * COUNT(*) / SUM(COUNT(*)) OVER (), 2) AS percentage
FROM `helpful-scion-492207-u1.Finance.Finance_enriched`
GROUP BY risk_profile;

#chi rischia evita FD?
SELECT
    AVG(CASE WHEN equity_market > 5 THEN fixed_deposits END) AS fd_when_equity_high,
    AVG(CASE WHEN equity_market <= 5 THEN fixed_deposits END) AS fd_when_equity_low
FROM `helpful-scion-492207-u1.Finance.Finance_enriched`;


Key findings
-Gli investitori più giovani mostrano una maggiore propensione al rischio
-Gli investitori con orizzonte di lungo periodo allocano una quota maggiore in strumenti azionari
-Gli investitori più conservativi preferiscono depositi vincolati e obbligazioni
-Il profilo di rischio è fortemente correlato con l’obiettivo di investimento


Python analysis https://colab.research.google.com/drive/1LMINJsDHcCutr7Uo2o2GPU0aWciwd9P5?usp=sharing

