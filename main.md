---
title: Software Design for Linear Mixed Model Specification
pages: 10-20 pages
author:
- familyname: Tanaka
  othernames: Emi
  instno: 1
  shorten: E. Tanaka
  institute: The University of Sydney, Camperdown NSW 2008, Australia
  email: dr.emi.tanaka@gmail.com
  firstauthor: true
abstract: |
 A statistical model is a mathematical representation of often a simplified or idealised data-generating process. A particular type of statistical model that is widely used is linear mixed models (LMMs), also called multi-level, nested, hierarchical or panel data models. LMMs are used widely in a variety of disciplines, e.g. agriculture, ecology, econometrics, psychology and so on, owing to its flexbility of accounting for complex correlated strucures in the data. This flexibility, however, gave rise to a number of ways to specify the LMMs in order to fit it via an application software. In this paper we review the software design of LMM (and its special case linear model) specification, in particular with the use of symbolic model formulae, with focus on the LMM specification in `lme4` and `asreml` R-packages.
keyword: [linear model, linear mixed model, linear mixed-effects model, multi-level model, hierarchical model, panel data model, model specification, model formulae, symbolic model formulae, API]
support: Supported by R Consortium
bibliography: biblio.bib
biblio-style: authoryear-comp
bibtex: true
output: 
  bookdown::pdf_document2:
    template: template/springer_computer_science_proceedings_template.tex
    fig_caption: yes
    fig_height: 5
    fig_width: 8
    #includes:
    #  in_header: preamble.tex
    keep_tex: yes
    number_sections: yes
    citation_package: biblatex
  bookdown::html_document2:
    toc_float: yes
    toc: yes
---






# Introduction

Statistical models are mathematical formulation characterising often  simplified real world phenomena. The use of statistical models is ubiquitous in many data analysis. These models are fitted or trained computationally, often with practitioners using some readily available application software or software package. In practice, statistical models in its mathematical (or descriptive) representation would require translation to the right input argument to fit using an application software. The design of these input arguments (called application programming interface, API) can help ease the friction in fitting user's desired model and focus user's time on important tasks, e.g. interpreting or using the fitted model.

While there is abundant application software to fit a variety of statistical models, the API is largely inconsistent and often restrictive. E.g. in linear models, intercept may or may not be included by default; and random error is assumed to be identical and independently distributed (i.i.d) with no option to modify these assumption. These inconsistencies and restrictiveness in the API cause great friction to fit the user's desired models. Some efforts are made in this front such as by the `parsnip` package [@Kuhn2018] in the R language [@R2018] to implement a tidy unified interface to many preditive modelling functions (e.g. random forest, logistic regression, linear regressoin etc) and `scikit-learn` library [@scikit-learn] for machine learning in the python language [@van1995python] that provides consistent API across its modules [@sklearn_api]. There is, however, little effort on consistency or discussion for the software specification of linear mixed models (LMMs). 

LMMs (also called hierarchical, panel data, nested or multi-level models) are widely used across many disciplines (e.g. ecology, psychology, agriculture, finance etc) due to its flexibility to model complex, correlated structures in the data. This flexibility is primarily achieved by the random effects and its variance-covariance structure - it is this flexibility, however, that results in major difference in model specification. In R, arguably the most popular general purpose package in R to fit LMMs is `lme4` [@Bates2015]. Total downloads from RStudio CRAN mirror from `cranlogs` [@cranlog] indicate there were over 2 million downloads for `lme4` whilst other popular mixed model packages in the same year have less than half a million (e.g. `nlme`: 461,000; `rstan`: 248,000; `brms`: 107,000).

Many software applications fit a special case of LMMs, (e.g. factor-analytic mixed models) thus limiting the model a user can fit. 


Test



With over 10,000 citations and 5.3 million total downloads on CRAN, `lme4` [@Bates2015] is arguably the most popular R-package to fit LMMs. Another popular R-package that fit LMMs with flexible covariance structures is  `asreml` [@Butler2009], which wraps the proprietary software ASreml [@Gilmour2009] into the R framework. `asreml` with its core algorithm collectively has over 4,000 citations and remains popular, in particular for the analysis of plant breeding trials, due to the ease of fitting flexible covariance structures despite the license cost for its continued use. 

Symbolic model formulae define the structural component of a statistical model in an easier and often more accessible terms for practitioners. The earlier instance of symbolic model formulae for linear models was applied in Genstat [@genstat] with further generalisation by @Wilkinson1973. @Chambers1993 describe the symbolic model formulae implementation for linear models in the `S` language which remains much the same in the `R` language [@Venables2018].


While the symbolic formula of linear models generally have a consistent representation and evaluation rule as implemented in `stats::formula`, this is not the case for LMMs (and mixed models more generally). The inconsistency of symbolic formulae arises mainly in the representation of random effects, with the additional need to specify the variance-covariance structure of the random effects as well as structure of the associated model matrix that governs how the random effects are mapped to (groups of) the observational units. For example, `asreml::asreml` have separate formulation of fixed and random effects while `lme4::lmer` have a single formula that includes a mixture of fixed and random effects. Further differences with motivating examples are shown under The Plan. The differences give rise to confusion of equivalent model specification in different R-packages.  

The lack of consistency in symbolic formula and model representation across mixed model software motivates the need to formulate a unified symbolic model formulae for LMMs with: (1) extension of the evaluation rules described in @Wilkinson1973; and (2) ease of comprehension of the specified model for the user. This symbolic model formulae can be a basis for creating a common API to mixed models with wrappers to popular mixed model R-packages (with initial focus on `lme4` and `asreml`), thereby achieving a similar feat to `parsnip` R-package [@Kuhn2018] which implements a tidy unified interface to many predictive modelling functions (e.g. random forest, logistic regression, survival models etc). 

# Linear Models

A special case of linear mixed models is the linear (fixed) models where the model comprises of only fixed effects and a single random term (i.e. the error) given in matrix notation as 
$$\boldsymbol{y} = \mathbf{X}\boldsymbol{\beta} + \boldsymbol{e}$$
where $\boldsymbol{y}$ is a $n$-vector of response, $\mathbf{X}$ is the $n\times p$ design matrix for associated $p$-vector of fixed effects $\boldsymbol{\beta}$ and $\boldsymbol{e}$ is the $n$-vector of random error. Typically we assume $\boldsymbol{e} \sim N(\boldsymbol{0}, \sigma^2\mathbf{I}_n)$.

For linear models, the software specification is largely divided into two approaches: (1) input of arrays for the response $\boldsymbol{y}$ and design matrix for fixed effects $\mathbf{X}$ and (2) symbolic model formulae. 


Some including intercept by default (i.e. if there is no column of ones supplied in the design matrix, it will append it by default).

How the categorical variable is dealt.


In the software specification of linear models, the argument typically consists of a two-dimentional array with named columns - this can be in the form of a design matrix $\mathbf{X}$, along with a vector of response $\boldsymbol{y}$, or a data frame where the column names are used to represent the 


* Data: the data may be given in a form of array with named columns. 
\begin{description}
\item[Data] The data may be given in a form of array.
\begin{description}
\item[Intercept]
\item[Categorical data] Constraints. 
\item[Interactions]
\end{description}
\end{description}



For example, fitting linear models in python [@van1995python] `scikit-learn` library (imported as `sklearn`) using the `LinearRegression` class requires the user to input the response and the design matrix for the fixed effects to the `fit` function [see more about the wider API discussion in @sklearn_api]. 


## Symbolic Model Formulae

# Linear Mixed Models

Consider a $n$-vector of response $\boldsymbol{y}$ modelled as 
$$\boldsymbol{y} = \mathbf{X}\boldsymbol{\beta} + \mathbf{Z}\boldsymbol{u} + \boldsymbol{e}$$


# Discussion

Software packages that fit statistical models have varying input arguments. This inconsistency 
