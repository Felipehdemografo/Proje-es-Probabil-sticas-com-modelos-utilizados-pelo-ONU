# Projeções Probabilísticas com modelos utilizados pela ONU
Abaixo são apresentados os comandos para fazer a projeção das componentes demográficas e da população, baseado nos modelos bayesianos desenvolvidos pela equipe do Centro de Estatística e Ciências Sociais da Universidade de Washington. Esses modelos são utilizados nas projeções realizadas para o World Population Prospects (WPP) da ONU. 
# Tutorial de Projeções Demográficas Probabilísticas

Este tutorial apresenta, passo a passo, como realizar projeções probabilísticas dos componentes demográficos — **esperança de vida**, **TFT**, **migração** — e finalmente **projeções populacionais** utilizando os pacotes **bayesLife**, **bayesTFR**, **bayesMig** e **bayesPop**.

---

## 1. Instalação dos Pacotes Necessários
Esse pacote é necessário para instalar pacotes que estejam hospedados no github. Vale a ressalva que algumas funções só irão rodar corretamente com os pacotes mais
mais recentes que estão disponíves no Github.
```r
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
O pacote básico para a esperança de vida é o bayeslife
Ševcíková, H., & Raftery, A. E. (2011). bayesLife: Bayesian projection of life expectancy.
Lembrando que os autores da função indicam projetar a esperança de vida feminina e projetar a esperança de vida masculina por relação com a feminina.

### 2.1 Preparação

```r
rm(list=ls())
install.packages("bayesLife", dependencies = TRUE)
library(bayesLife)

getwd()
setwd("C:/Users/.../Dados")
```
Sempre lembrar de alterar esse diretório para o do seu computador.

```r
e0.dir <- "e0simulation_SC"
data.e0F <- "e0f_SC.txt"
```
O primeiro comando indica o diretório onde serão armazenados os resultados, já o segundo é a leitura da base de dados utilizada. Nesse caso está sendo fornecido um histórico de esperança de vida fora do pacote WPP. Aqui cabe uma observação muito importante, a forma com que o arquivo é lido é essencial para o funcionamento correto da função
O banco de dados tem o formato abaixo, esse é exatamente o banco de dados que utilizamos na função.

country_code	country_name	reg_code	SIGLA	geocode	region	1980-1985	1985-1990	1990-1995	1995-2000	2000-2005	2005-2010	2010-2015	2015-2020	2020-2025	last.obs	first.obs	include_code
76		        Brazil    		440    		SC  	7600440	States:SC	70.53		72.82	  	74.91  		76.12  		77.70  		78.75	  	79.44  		80.35  		79.73  		2025			1980			2

Sobre o banco de dados da esperança de vida, é necessário identificar o país, código e nome, depois o estado, código e nome, depois uma série histórico da esperança de vida
da região/país estudado. Ao final da série histórica há uma coluna informando qual o último ano da série (last.observed) e o primeiro ano da série (first.observed).
Essa estrutura vale para os dados das 3 componentes. Os dados das componentes são do quinquenio e pode ser adotado os dados do ano do meio do período de 5 anos,
por exemplo 1980-1985 é a esperança de vida do meio do ano de 1982. Ou ainda uma média dos anos do período. Lembrando que os dados podem ser anuais.
O ideal é que a função tenha uma quantidade maior de iterações, inclusive é possível executar a função para que ela faça iterações até a convergência, mas vale a 
observação que quanto maior a quantidade de iterações mais tempo levará. Para testes ou execuções com com finalidade de aprendizagem recomendamos 1000 iterações.
No primeiro momento (função "run..mcm") é ajustado o modelo para definir os parametros e no segundo (função ".predict") que vem mais a frente o modelo será utilizado para projetar. Tanto a observação das iterações, quanto a do ajuste vale para as 3 componentes demográficas.



### 2.2 Estimação dos Parâmetros (run.e0.mcmc)

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
