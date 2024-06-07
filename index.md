---
title: Online gehostete Anweisungen
permalink: index.html
layout: home
---

Dieses Repository enthält die Übungen für das Praxislab zum Microsoft-Kurs [DP-420: Entwerfen und Implementieren von cloudnativen Anwendungen mit Microsoft Azure Cosmos DB][course-description] und den zugehörigen [Modulen zur eigenverantwortlichen Bearbeitung in Microsoft Learn][learn-collection]. Diese Übungen begleiten die Lernmaterialen und erleichtern die praktische Anwendung der beschriebenen Technologien.

> &#128221; Sie benötigen ein Microsoft Azure-Abonnement für diese Übungen. Sie können sich unter [https://azure.microsoft.com][azure] für eine kostenlose Testversion registrieren.

## Labs

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %}
| Modul | Lab |
| --- | --- |
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

[azure]: https://azure.microsoft.com
[course-description]: https://docs.microsoft.com/learn/certifications/courses/dp-420t00
[learn-collection]: https://docs.microsoft.com/users/msftofficialcurriculum-4292/collections/1k8wcz8zooj2nx
