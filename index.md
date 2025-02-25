---
title: Online gehostete Anweisungen
permalink: index.html
layout: home
---

Dieses Repository enthält die Übungen für das Praxislab zum Microsoft-Kurs [DP-420: Entwerfen und Implementieren von cloudnativen Anwendungen mit Microsoft Azure Cosmos DB][course-description] und den zugehörigen [Modulen zur eigenverantwortlichen Bearbeitung in Microsoft Learn][learn-collection]. Diese Übungen begleiten die Lernmaterialen und erleichtern die praktische Anwendung der beschriebenen Technologien.

> &#128221; Sie benötigen ein Microsoft Azure-Abonnement für diese Übungen. Sie können sich unter [https://azure.microsoft.com][azure] für eine kostenlose Testversion registrieren.

# Labs

## C#-Labs

{% assign labs = site.pages | where_exp:"page", "page.url contains '/instructions'" %} {% assign csharp_setup_labs = "" | split: "," %} {% assign csharp_regular_labs = "" | split: "," %} {% assign genai_setup_labs = "" | split: "," %} {% assign genai_python_labs = "" | split: "," %} {% assign genai_javascript_labs = "" | split: "," %}

{% for activity in labs %} {% assign segments = activity.url | split: "/" %}

  {% if segments[1] == "instructions" and segments.size == 3 %} {% if activity.lab.module contains "Setup" %} {% assign csharp_setup_labs = csharp_setup_labs | push: activity %} {% else %} {% assign csharp_regular_labs = csharp_regular_labs | push: activity %} {% endif %}
  
  {% elsif activity.url contains '/gen-ai/python/instructions' %} {% assign genai_python_labs = genai_python_labs | push: activity %}
  
  {% elsif activity.url contains '/gen-ai/javascript/instructions' %} {% assign genai_javascript_labs = genai_javascript_labs | push: activity %}
  
  {% elsif activity.url contains '/gen-ai/common/instructions' and activity.lab.module contains "Setup" %} {% assign genai_setup_labs = genai_setup_labs | push: activity %} {% endif %} {% endfor %}

---

### **Setup-Labs**

| Modul | Lab |
| --- | --- |
{% for activity in csharp_setup_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **Labs**

| Modul | Lab |
| --- | --- |
{% for activity in csharp_regular_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

## **Generative KI-Labs**

### **Allgemeine Setup-Labs**

| Modul | Lab |
| --- | --- |
{% for activity in genai_setup_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **JavaScript-Labs**

| Modul | Lab |
| --- | --- |
{% for activity in genai_javascript_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **Python-Labs**

| Modul | Lab |
| --- | --- |
{% for activity in genai_python_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

[azure]: https://azure.microsoft.com
[course-description]: https://docs.microsoft.com/learn/certifications/courses/dp-420t00
[learn-collection]: https://docs.microsoft.com/users/msftofficialcurriculum-4292/collections/1k8wcz8zooj2nx
