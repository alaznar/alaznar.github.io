---
layout: splash
title: "Alejandro Aznar - Data Scientist"
subtitle: "R · Python · Power BI · Automation · Safety/Clinical Data"
header:
  overlay_filter: 0.2
  overlay_image: /assets/img/header.jpg # opcional
  actions:
    - label: "Descargar CV"
      url: /assets/CV_Alejandro_Aznar.pdf
    - label: "LinkedIn"
      url: https://www.linkedin.com/in/alejandroaznar-/
intro:
  - excerpt: >
      **Propuesta de valor.** Data Scientist with experience in Safety and Clinical Data. R, Python and Power BI.
feature_row:
  - image_path: /assets/img/Garmin_teaser.png
    alt: "Garmin Data Analysis"
    title: "Garmin Data"
    excerpt: "This project is an end-to-end personal Garmin data analysis combining exploration, visualization, and modeling. Data was cleaned and merged using Pandas and NumPy, with EDA performed through Matplotlib and Seaborn (distributions, correlations, pairplots) and GPS routes mapped via gpxpy + Folium. Linear regression models were built to estimate calories/pace from running metrics, applying log transformations, imputation, and scaling, while validating assumptions with Shapiro-Wilk, Breusch-Pagan, QQ-plots, and VIF. Additionally, K-Means clustering (with k selection via silhouette score) was used to segment training sessions and patterns. The result was a hybrid data visualization and data science project with interpretable models, delivering actionable insights into training and recovery habits."
    url: /projects/garmin/
    btn_label: "See Project"
    btn_class: "btn--primary"
  - image_path: /assets/img/Diabetes_teaser.png
    alt: "Diabetes Multiclass Prediction"
    title: "Multiclass Prediction of Diabetes"
    excerpt: "Classification of Diabetic classes by deploying ML algorithms"
    url: /projects/diabetes/
    btn_label: "See Project"
    btn_class: "btn--primary"
---

{% include feature_row id="intro" type="center" %}

{% include feature_row %}
