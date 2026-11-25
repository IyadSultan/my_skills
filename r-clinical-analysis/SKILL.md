---
name: r-clinical-analysis
description: "R Markdown template and best-practice patterns for clinical/oncology research analysis. Use when writing R code for medical datasets, survival analysis, GvHD studies, BMT outcomes, or any clinical report requiring baseline characteristics tables, Kaplan-Meier curves, univariable/multivariable Cox regression, and publication-ready output. Covers data cleaning, date handling, factor conversion, gtsummary tables, finalfit Cox regression, survminer plots with Lancet theming, and data export."
---

# R Clinical Analysis Skill

*Best-practice patterns for clinical research R Markdown reports — data cleaning through survival analysis and publication-ready tables.*

---

## Core Principles

1. **Always use `dplyr::count`, `dplyr::recode`, `dplyr::select`** — namespace-qualify these calls
2. **Always use `cmprsk::cuminc`** for competing-risks cumulative incidence
3. **Never `print()` a `gt`, `tbl_summary`, or `gtsummary` object** — render with `gt::gt()` or let knitr auto-print
4. **Use `gt::gt()` to display `finalfit` tables**, not `kable()`
5. **All character columns → factors** before modeling
6. **White-on-white email rendering bug**: always set `bgcolor` on table elements when generating HTML emails
7. **Lancet colour scheme** via `ggsci::pal_lancet("lanonc")` and `scale_color_lancet()` / `scale_fill_lancet()`
8. **prepare first an analytic only text/numbers with no figures notebook.  Then I will show you the results for yout to prepare all figures and tables.  9. Always present survial summary with estimates and +/-SE at 1,3,5 years below survival figures. Use theme_classic() for survival curves and do not show risk tables.
10. Use  pacman package to import libraries.  

---

## Document Setup (always start here)

````r
---
title:  "Choose a Descriptive Title"
output: html_document
date: "`r Sys.Date()`"
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(
  echo    = FALSE,
  message = FALSE,
  warning = FALSE
)

pacman::p_load(
  dplyr, tidyr, lubridate, summarytools, gtsummary, Hmisc, here,
  ggplot2, janitor, stringr, survival, survminer, finalfit, ggsci,
  gt, gtExtras, knitr, kableExtra, cmprsk
)

here::set_here()

# Theme and colours
theme_set(theme_minimal())
lancet_colors <- ggsci::pal_lancet("lanonc")(9)
```
````

---

## Running & Knitting R from the Terminal

This machine can run R and knit `.Rmd` directly from the shell — no RStudio GUI needed. **Render from the command line, don't ask the user to click Knit.**

### Render an Rmd to its output format

```bash
# Always quote the path — project dirs here often contain spaces
Rscript -e 'rmarkdown::render("manuscript/manuscript.Rmd")'
```

`rmarkdown::render()` auto-installs CRAN packages declared in the setup chunk's
`pacman::p_load(...)` / `install.packages(missing)` block on first run, so a fresh
`.Rmd` usually knits without manual package installs. If `requireNamespace` reports
something missing, install explicitly:

```bash
Rscript -e 'install.packages(c("meta","metafor"), repos="https://cloud.r-project.org")'
```

### pandoc (required by `rmarkdown::render`) — already wired up

Pandoc is **not** on `PATH` as a standalone binary on this machine, but RStudio's
bundled pandoc 3.8.3 is used instead. The path is persisted in `~/.Renviron`:

```
RSTUDIO_PANDOC="/Applications/RStudio.app/Contents/Resources/app/quarto/bin/tools/aarch64"
```

This takes effect in every **new** R session, so `rmarkdown::pandoc_available()` returns
`TRUE` automatically. If a render ever fails with "pandoc version 1.12.3 or higher is
required", that file was lost — re-add the line above (use `tools/x86_64` on Intel Macs),
or `brew install pandoc` for a standalone copy independent of RStudio.

### `here::here()` path gotcha (important)

When a project has **no** root marker (no `.Rproj`, no `.git`), `here::here()` resolves to
the **Rmd's own folder** during knit (e.g. `manuscript/`), so `here::here("data", ...)`
wrongly looks in `manuscript/data/`. Fix once by dropping an empty anchor file at the true
project root:

```bash
touch .here   # at the project root, NOT the Rmd's folder
```

Alternatively render with an explicit root:

```bash
Rscript -e 'rmarkdown::render("manuscript/manuscript.Rmd", knit_root_dir=getwd())'
```

### Quick smoke test of the toolchain

```bash
Rscript -e 'cat("pandoc:", rmarkdown::pandoc_available(),
  as.character(rmarkdown::pandoc_version()), "\n")'
```

### Run analysis code without knitting

For iterating on logic, run a plain `.R` script or a one-liner — faster than a full knit:

```bash
Rscript path/to/analysis.R
Rscript -e 'source("R/01_clean.R"); str(ds)'
```

---

## Data Cleaning Pipeline

### 1 · Remove Sensitive Columns

```r
sensitive_patterns <- c(
  "name", "mrn", "medical_record", "patient_id", "subject_id",
  "^id$", "_id$", "identifier", "ssn", "address", "phone", "email"
)

sensitive_cols <- names(ds)[
  stringr::str_detect(tolower(names(ds)),
                      paste(sensitive_patterns, collapse = "|"))
]

ds <- ds |> dplyr::select(-dplyr::all_of(sensitive_cols))
```

### 2 · Clean Column Names

```r
original_cols <- names(ds)
ds <- ds |> janitor::clean_names()
```

### 3 · Standardise Text

```r
ds <- ds |>
  dplyr::mutate(dplyr::across(where(is.character), stringr::str_trim)) |>
  dplyr::mutate(dplyr::across(where(is.character), stringr::str_squish)) |>
  dplyr::mutate(dplyr::across(where(is.character), ~ dplyr::na_if(., "")))
```

### 4 · Parse Dates

```r
# Update date_format to match your data: "%Y-%m-%d", "%m/%d/%Y", "%d/%m/%Y", "%d-%b-%Y"
date_format <- "%Y-%m-%d"

date_cols <- names(ds)[
  stringr::str_detect(tolower(names(ds)),
                      "date|time|diagnosis|death|progression|relapse|follow|fu|last|birth")
]

for (col in date_cols) {
  ds[[col]] <- as.Date(ds[[col]], format = date_format)
}
```

### 5 · Convert Characters to Factors

```r
ds <- ds |>
  dplyr::mutate(dplyr::across(where(is.character), as.factor))
```

### 6 · Post-cleaning Summary Table (always provide)

```r
# Show before/after examples for key variables, especially dates
cleaning_example <- data.frame(
  Variable = c("date_of_diagnosis", "gender", "mrn"),
  Before   = c("15/01/2015", "MALE ", "123456"),
  After    = c("2015-01-15", "male",  "[removed]")
)
gt::gt(cleaning_example)
```

---

## Data Summary

### dfSummary

```r
df_summary <- summarytools::dfSummary(
  ds,
  plain.ascii  = FALSE,
  style        = "grid",
  graph.magnif = 0.75,
  valid.col    = FALSE,
  tmp.img.dir  = "/tmp"
)
print(df_summary, method = "render")
```

### Overall gtsummary Table

```r
ds |>
  gtsummary::tbl_summary(
    statistic = list(
      all_continuous()  ~ "{mean} ({sd})",
      all_categorical() ~ "{n} ({p}%)"
    ),
    missing_text = "Missing"
  ) |>
  gtsummary::modify_header(label ~ "**Variable**") |>
  gtsummary::modify_caption("**Table 1. Patient Characteristics**") |>
  gtsummary::bold_labels()
```

### Stratified gtsummary Table (Table 1)

```r
# Replace `treatment_group` with your stratification variable
ds |>
  gtsummary::tbl_summary(
    by = treatment_group,
    statistic = list(
      all_continuous()  ~ "{mean} ({sd})",
      all_categorical() ~ "{n} ({p}%)"
    ),
    missing_text = "Missing"
  ) |>
  gtsummary::add_p() |>
  gtsummary::modify_header(label ~ "**Variable**") |>
  gtsummary::modify_caption("**Table 2. Characteristics by Treatment Group**") |>
  gtsummary::bold_labels()
```

---

## Survival Data Preparation

```r
ds <- ds |>
  dplyr::mutate(
    # Overall Survival (months)
    os_months = as.numeric(
      difftime(death_date, date_of_diagnosis, units = "days")
    ) / 30.44,

    # Event-Free Survival: earliest of death or progression
    efs_event_date = pmin(death_date, progression_date, na.rm = TRUE),
    efs_months = as.numeric(
      difftime(efs_event_date, date_of_diagnosis, units = "days")
    ) / 30.44,

    # Event indicators (1 = event, 0 = censored)
    os_event  = dplyr::if_else(is.na(death_date), 0L, 1L),
    efs_event = dplyr::if_else(is.na(efs_event_date), 0L, 1L),

    # Use last follow-up for censored observations
    os_months = dplyr::if_else(
      is.na(os_months),
      as.numeric(difftime(date_last_fu, date_of_diagnosis, units = "days")) / 30.44,
      os_months
    ),
    efs_months = dplyr::if_else(
      is.na(efs_months),
      as.numeric(difftime(date_last_fu, date_of_diagnosis, units = "days")) / 30.44,
      efs_months
    )
  )
```

---

## Kaplan-Meier Plots (Lancet Theme)

### Single-Curve (OS + EFS Combined)

```r
# Shared plot parameters
FONT    <- 12
BREAKS  <- 12        # x-axis break interval (months)
MO      <- 60        # x-axis maximum (months)
XLAB    <- "Time (months)"
YLAB    <- "Probability"
conf_int <- FALSE
REVERSE  <- FALSE

os  <- survfit(Surv(os_months,  os_event)  ~ 1, data = ds)
efs <- survfit(Surv(efs_months, efs_event) ~ 1, data = ds)

plots <- list(OS = os, EFS = efs)

FIT <- ggsurvplot(
  plots,
  data         = ds,
  combine      = TRUE,
  text         = FONT,
  legend       = c(0.8, 0.8),
  legend.title = "",
  legend.labs  = c("Overall Survival", "Event-Free Survival"),
  surv.scale   = "percent",
  break.x.by  = BREAKS,
  conf.int    = conf_int,
  xlim        = c(0, MO),
  axes.offset = FALSE,
  xlab        = XLAB,
  ylab        = YLAB
)

FIG <- FIT$plot +
  scale_color_lancet() +
  scale_fill_lancet() +
  theme(
    axis.text.x  = element_text(size = FONT),
    axis.text.y  = element_text(size = FONT),
    axis.title   = element_text(size = FONT),
    legend.text  = element_text(size = FONT),
    plot.margin  = unit(c(1, 1, 1, 1), "lines")
  ) +
  guides(colour = guide_legend(reverse = REVERSE))

print(FIG)
```

### Stratified Curve (by Group)

```r
# Replace `treatment_group` with your grouping variable
fit_os_strat <- survfit(Surv(os_months, os_event) ~ treatment_group, data = ds)

ggsurvplot(
  fit_os_strat,
  data         = ds,
  conf.int     = TRUE,
  pval         = TRUE,
  risk.table   = TRUE,
  palette      = "lancet",
  title        = "Overall Survival by Treatment Group",
  xlab         = "Time (months)",
  ylab         = "Survival Probability",
  break.time.by = 12,
  legend.title = "Treatment",
  ggtheme      = theme_minimal(),
  tables.theme = theme_minimal()
)
```

---

## Cox Regression with finalfit + gt

### How to Display Tables

**Always use `gt::gt()` — never `kable()` or `print()` on the raw object.**

```r
# Define dependent and explanatory variables
dependent_os  <- "Surv(os_months, os_event)"
dependent_efs <- "Surv(efs_months, efs_event)"

explanatory_vars <- c(
  "age_yr", "gender", "diagnosis_group",
  "conditioning_intensity", "pre_bmt_disease_status"
)

# Overall Survival
os_cox_table <- ds |>
  finalfit::finalfit(dependent_os, explanatory_vars, metrics = TRUE)

cat("### Overall Survival — Cox Regression")
gt::gt(os_cox_table[[1]])

# Event-Free Survival
efs_cox_table <- ds |>
  finalfit::finalfit(dependent_efs, explanatory_vars, metrics = TRUE)

cat("### Event-Free Survival — Cox Regression")
gt::gt(efs_cox_table[[1]])
```

---

## Competing Risks (cmprsk)

```r
# Always namespace-qualify cuminc
ci_fit <- cmprsk::cuminc(
  ftime   = ds$time_to_event,
  fstatus = ds$event_type,   # 1 = event of interest, 2 = competing event, 0 = censored
  group   = ds$treatment_group
)

plot(ci_fit,
     xlab  = "Time (months)",
     ylab  = "Cumulative Incidence",
     col   = lancet_colors[1:4],
     lwd   = 2,
     main  = "Cumulative Incidence by Treatment Group")
```

---

## Variable Counts (always namespace-qualify)

```r
# Always dplyr::count, never count()
ds |> dplyr::count(treatment_group, sort = TRUE)
ds |> dplyr::count(diagnosis_group, treatment_group)

# Always dplyr::select
ds |> dplyr::select(age_yr, gender, treatment_group, os_months, os_event)

# Always dplyr::recode
ds <- ds |>
  dplyr::mutate(
    risk_group = dplyr::recode(risk_raw,
      "low"  = "Low Risk",
      "int"  = "Intermediate Risk",
      "high" = "High Risk"
    )
  )
```

---

## Data Export and Session Info

```r
# Export cleaned dataset
write.csv(ds, "cleaned_data.csv", row.names = FALSE)

cat(paste("Final dataset:", nrow(ds), "rows ×", ncol(ds), "columns\n"))
sessionInfo()
```

---

## Anti-Patterns to Avoid

```r
# WRONG — unqualified calls
count(ds, treatment_group)        # Use dplyr::count(ds, treatment_group)
select(ds, age, gender)           # Use dplyr::select(ds, age, gender)
recode(x, "a" = "b")             # Use dplyr::recode(x, "a" = "b")
cuminc(ftime, fstatus)           # Use cmprsk::cuminc(ftime, fstatus)

# WRONG — printing gt/gtsummary objects
print(tbl_summary(...))           # Just call tbl_summary(...) or wrap in gt::gt()
print(gt_object)                  # Call gt::gt(df) directly

# WRONG — kable for finalfit output
kable(finalfit_table[[1]])        # Use gt::gt(finalfit_table[[1]])

# WRONG — not using Lancet palette
ggsurvplot(..., palette = "hue")  # Use palette = "lancet" or scale_color_lancet()
```

---

## Checklist for Every Analysis

- [ ] Run `str(ds)`, `names(ds)`, `head(ds, 3)`, `dim(ds)` first
- [ ] Confirm date format and parse correctly
- [ ] Remove sensitive columns (MRN, name, ID)
- [ ] Provide before/after cleaning example table
- [ ] Convert all character columns to factors
- [ ] Specify diagnosis date, death date, last follow-up date, event status
- [ ] Choose explanatory variables for multivariable Cox
- [ ] Choose stratification variable for Table 1 and KM curves
- [ ] Verify `dplyr::count`, `dplyr::select`, `dplyr::recode`, `cmprsk::cuminc` are namespace-qualified
- [ ] Display all regression tables with `gt::gt()`
- [ ] Export cleaned dataset