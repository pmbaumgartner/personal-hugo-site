---
title: "Tableau and Raw Data as Sneaky Technical Debt"
date: 2019-07-30T13:27:47-04:00
draft: false
---

We work on a lot of projects where we need to iterate quickly on visualizations generated from a flat file of data. Tableau is a great tool to get started with this, but its ease of use can quickly lead people towards illusions of progress, misunderstandings of the underlying data, or technical debt.

Usually this comes up in a conversation like this:

> **Them**: I want to build this specific type of chart that represents this information. I've tried `<a combination of parameters, measure names or measure values, and calculated fields>` and I just can't get it to work.
> 
> **Me**: What does your underlying data look like?
> 
> **Them**: Why does that matter? Isn't Tableau supposed to be _so easy_ I can generate whatever chart I want? Can't I use their data manipulation tool to get what I need? I'm 90% of the way there with these 4 calculated fields and this parameter. Plus, I can't go back and change the data, this is the format given by the client.

**The problem is**: the visualization developer has found effective traction on low-hanging fruit by creating visualizations that are easily facilitated with the existing data structure. However, now that they're trying to answer the business or research questions driving the need for visualization in the first place and they're stuck and frustrated because the data isn't setup correctly. This can evolve into a kind of [technical debt](https://en.wikipedia.org/wiki/Technical_debt): the longer a user puts off structuring the data in a way that aligns with the business problem, the harder it is to continue building visualizations and dashboards in an efficient manner.

**The solution is:**

1. **Notice when you have an abundance of Calculated fields or Parameters that are operating heavily as data manipulation tools.** If the logic of these fields is to manipulate the data into a format to get a chart you want, rather than derive a new variable or do something simple across variables with a parameter, you might need to change your data structure.
2. **Notice if most of the time you're using Measure Values or Measure Names in your charts.** Typically this happens when you want to visualize data across columns in your original dataset.
3. **Melt or pivot your column-oriented data to being row-oriented. [Tidy data](https://vita.had.co.nz/papers/tidy-data.pdf) principles can get you quite far here.** However, your specific problem means you'll need to think about your data and the semantic and logical structure of it. You probably can't just melt across all columns; some columns might have mutually derived data. Finally, consider how your future aggregations in the dashboard will change to handle your future data. If you're going from wide-to-long, often you'll need to take advantage of `COUNT DISTINCT` because you have duplicate rows for each of your original records.

Being able to quickly generate good-looking visualizations is a helpful skill, but don't let a lust for ease blind you to the always necessary alignment between data structure and business problem.
