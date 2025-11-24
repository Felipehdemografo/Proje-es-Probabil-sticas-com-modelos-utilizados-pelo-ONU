# Projeções Probabilísticas com modelos utilizados pela ONU
Abaixo são apresentados os comandos para fazer a projeção das componentes demográficas e da população, baseado nos modelos bayesianos desenvolvidos pela equipe do Centro de Estatística e Ciências Sociais da Universidade de Washington. Esses modelos são utilizados nas projeções realizadas para o World Population Prospects (WPP) da ONU. 
# Tutorial de Projeções Demográficas Probabilísticas

Este tutorial apresenta, passo a passo, como realizar projeções probabilísticas dos componentes demográficos — **esperança de vida**, **TFT**, **migração** — e finalmente **projeções populacionais** utilizando os pacotes **bayesLife**, **bayesTFR**, **bayesMig** e **bayesPop**.

---

## 1. Instalação dos Pacotes Necessários
Esse pacote é necessário para instalar pacotes que estejam hospedados no github. Vale a ressalva que algumas funções só irão rodar corretamente com os pacotes mais
mais recentes que estão disponíves no Github.
```r
#Pacotes necessários
install.packages("devtools", dependencies = TRUE)
library(devtools)
options(timeout = 600)

install_github("PPgp/wpp2019")
library(wpp2019)
install_github("PPgp/bayesPop")
library(bayesPop)
install_github("PPgp/bayesLife")
library(bayesLife)
install_github("PPgp/bayesTFR")
library(bayesTFR)
install_github("PPgp/bayesMig")
library(bayesMig)
install_github("PPgp/MortCast")
library(MortCast)
```

---

## 2. Projeção da Esperança de Vida (bayesLife)

### 2.1 Preparação

```r
rm(list=ls())
install.packages("bayesLife", dependencies = TRUE)
library(bayesLife)

getwd()
setwd("C:/Users/.../Dados")
```

### 2.2 Estimação dos Parâmetros (run.e0.mcmc)

```r
e0.dir <- "e0simulation_SC"
data.e0F <- "e0f_SC.txt"

me0_Brasil <- run.e0.mcmc(
  my.e0.file=data.e0F,
  output.dir=e0.dir,
  iter=1000,
  nr.chains=2,
  thin=1,
  verbose.iter=20,
  replace.output=TRUE,
  start.year=1980,
  present.year=2020
)
```

### 2.3 Projeção da Esperança de Vida

```r
e0.pred <- e0.predict(sim.dir=e0.dir, end.year=2070,
                      replace.output=TRUE, burnin=500, nr.traj=1000)
```

---

## 3. Projeção da TFT (bayesTFR)

### 3.1 Preparação

```r
rm(list=ls())
install.packages("bayesTFR", dependencies = TRUE)
library(bayesTFR)
setwd("C:/Users/.../Dados")
```

### 3.2 Estimação da Logística da TFT

```r
tfr.dir <- "TFRsimulation_SC"
data.tfr <- "TFT_SC.txt"

m2 <- run.tfr.mcmc(
  my.tfr.file=data.tfr,
  output.dir=tfr.dir,
  iter=1000,
  nr.chains=2,
  thin=1,
  verbose.iter=20,
  replace.output=TRUE,
  start.year=1980,
  present.year=2020,
  use.wpp=FALSE
)
```

### 3.3 Estimação Fase 3

```r
m3 <- run.tfr3.mcmc(
  my.tfr.file=data.tfr,
  sim.dir=tfr.dir,
  iter=1000,
  replace.output=TRUE,
  nr.chains=2,
  thin=2
)
```

### 3.4 Projeção da TFT

```r
tfr.pred <- tfr.predict(sim.dir=tfr.dir, end.year=2070,
                        burnin=30, burnin3=25, nr.traj=1000,
                        replace.output=TRUE)
```

---

## 4. Projeção da Migração (bayesMig)

### 4.1 Preparação

```r
setwd("C:/Users/.../Dados")

mig.dir <- "migracao_simulation_SC"
data.migF_Brasil <- "taxa_liquida_migracao_estados.txt"
```

### 4.2 Estimação dos Parâmetros

```r
mig_Brasil <- run.mig.mcmc(
  my.mig.file=data.migF_Brasil,
  output.dir=mig.dir,
  iter=1000,
  nr.chains=2,
  thin=1,
  verbose.iter=20,
  replace.output=TRUE,
  start.year=1980,
  present.year=2020
)
```

### 4.3 Projeção da Migração

```r
mig.pred <- mig.predict(
  sim.dir=mig.dir,
  end.year=2070,
  replace.output=TRUE,
  burnin=500,
  nr.traj=1000
)
```

---

## 5. Projeção da População (bayesPop)

### 5.1 Preparação

```r
rm(list=ls())
library(bayesPop)
setwd("C:/Users/.../Dados")
```

### 5.2 Definição dos Diretórios

```r
tfr.dir <- "TFRsimulation_RR"
e0.dir <- "e0simulation_RR"
pop.dir <- "POPsimulation_RR_teste"
mig.dir <- "migracao_simulation_RR_internacional"
```

### 5.3 Rodando o Modelo Final (pop.predict)

```r
pop.pred <- pop.predict(
  end.year=2070,
  start.year=1980,
  present.year=2020,
  output.dir=pop.dir,
  nr.traj=1000,
  inputs=list(
    tfr.sim.dir=tfr.dir,
    e0F.sim.dir=e0.dir,
    e0M.sim.dir="joint_",
    popM=popM_Brasil,
    popF=popF_Brasil,
    mxM=mxM_Brasil,
    mxF=mxF_Brasil,
    migtraj=migration_Brasil
  ),
  keep.vital.events=TRUE,
  lc.for.all=FALSE,
  replace.output=TRUE
)
```

### 5.4 Análise dos Resultados

```r
country <- "Brazil"
summary(pop.pred, country=country)
pop.trajectories.table(pop.pred, country=country, pi=c(80,95))
pop.byage.table(pop.pred, country=country, pi=c(80,90), year=2025, sex="female")
pop.trajectories.plot(pop.pred, country=country, nr.traj=30)
pop.pyramid(pop.pred, country, year=c(2060,2010))
```

---
