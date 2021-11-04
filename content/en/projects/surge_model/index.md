---
title: "COVID-19 Mental Health Surge Model"
weight: 1
resources:
    - src: app_01.png
      params:
          weight: -100
          
    - src: app_02.png
      params:
          weight: -100
project_timeframe: June-October 2020
---

The [Stategy Unit][SU] were commissioned by [Mersey Care NHS FT][MC] to create a model to estimate the surge in demand
to Mental Health services caused by the COVID-19 pandemic. Based on published evidence, [SUS] datasets ([MHSDS][MHSDS],
[IAPT][IAPT]), and expert opinion we produced a System's Dynamic's Model in R. This model was then given a web user
interface so people could interact with the model.

The code is released under an MIT license and is available on the [SU GitHub][GH], and the model is available to use
[here][model].

[SU]: https://strategyunitwm.nhs.uk/
[MC]: https://www.merseycare.nhs.uk/
[SUS]: https://digital.nhs.uk/services/secondary-uses-service-sus
[MHSDS]: https://digital.nhs.uk/data-and-information/data-collections-and-data-sets/data-sets/mental-health-services-data-set
[IAPT]: https://digital.nhs.uk/data-and-information/data-collections-and-data-sets/data-sets/improving-access-to-psychological-therapies-data-set
[GH]: https://github.com/The-Strategy-Unit/723_mh_covid_surge_modelling
[model]: https://strategyunit.shinyapps.io/MH_Surge_Modelling/
