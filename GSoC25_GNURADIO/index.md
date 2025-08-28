---
layout: home
title: "GSoC 2025 - GNU Radio 4.0 Block Set Expansion"
---

# GSoC 2025 - GNU Radio 4.0 Block Set Expansion

Welcome to my Google Summer of Code 2025 project documentation!

This project focuses on **Expanding the GNU Radio 4.0 Block Set** by porting and modernizing the most-used analog & digital signal-processing blocks from GNU Radio 3.x to the brand-new GNU Radio 4.0 architecture.

---

## 📋 **Project Documentation**

### **[📊 Final Report](gsoc-final-report.html)**
Comprehensive project summary, technical contributions, and outcomes

### **[📚 Porting Guide](porting-guide.html)**
Complete technical guide for porting GNU Radio 3.x blocks to 4.0

### **[📖 Porting Guide (Outline)](porting-guide-outline.html)**
Structured outline version of the porting guide

---

## 📝 **Progress Blog Posts**

{% for post in site.posts %}
  <div style="margin-bottom: 1rem;">
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <small>{{ post.date | date: "%B %d, %Y" }}</small>
    <p>{{ post.excerpt }}</p>
  </div>
{% endfor %}

---

## 🔗 **Project Links**

- **GitHub Repository**: [fair-acc/gnuradio4](https://github.com/fair-acc/gnuradio4)
- **Mentor**: Josh Morman
- **Organization**: GNU Radio
- **Duration**: May 2025 - August 2025

---

**[← Back to Main Portfolio](../)**
