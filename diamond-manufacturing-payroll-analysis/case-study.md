# How I Untangled a Mess of ERP Data and Found a Five-Figure Monthly Loss

**TL;DR:** I cleaned and standardized messy ERP operational data, rebuilt payroll calculations from scratch, identified a legacy logic error after a business model shift, and helped fix system and process gaps that were causing a recurring five-figure monthly loss.

## What I was working on

I was part of a small internal audit at a diamond manufacturing company. Three of us total—two internal people and a freelance forensic auditor. They needed someone to dig into the data side of things, specifically two big questions: Is our operational data even usable? And are we paying people correctly?

The company runs on a per-piece pay model. Workers move diamonds through different departments and processes, getting paid based on carat weight (CTS) and what work they're doing. All of this is supposed to be tracked and calculated automatically by their ERP.

 

## The data was a disaster

Everything came out of the ERP as Excel (XLS) files, but calling them “Excel files” almost felt misleading. These weren’t clean, table-structured exports. They were closer to formatted reports dumped into Excel, with inconsistent layouts and broken structure.

Some files didn’t even have proper headers.
Column names were split across multiple rows, with one column title starting in one row and the rest of the header continuing in the next. In some reports, the actual data started a few rows below the header, with blank rows or summary text in between. In others, the first few rows contained company names, dates, or report descriptions instead of data.

In a few cases, the same report would export differently depending on the date range selected—column positions would shift, extra columns would appear, or headers would slightly change wording. This made it impossible to assume a fixed schema.

Working with these XLS files in Excel was frustrating and error-prone.
Simple operations like filtering or removing duplicates required manually fixing headers first. If you missed even one misaligned column or blank row, formulas and filters would silently break. Large files were slow, and manual cleanup introduced the risk of accidental changes.

Importing the same files into Python added another layer of difficulty.
Because the headers weren’t consistent or properly aligned, pandas often couldn’t infer the correct column names. Some columns were read as Unnamed, others shifted under the wrong header, and data types were unreliable. Before any real analysis, I had to:

- Manually identify where the actual data started

- Reconstruct headers into a single, clean row

- Drop summary rows and non-data content

- Standardize column names across files

Only after doing this could I treat the files as actual datasets instead of semi-structured reports.

But even after fixing the structure, the content itself was still messy.

Worker names were completely inconsistent. “Ramesh Kumar” in one report, “R Kumar” in another, “Kumar Ramesh” somewhere else, “Ramesh Kuamr” (typo) in a fourth place. Sometimes just initials. Sometimes nicknames. No employee IDs, no consistent format, nothing.

Process names were just as bad. “Polishing,” “polishing,” “Polish,” “Polshing” (misspelled), “Pol” (abbreviated)—all referring to the same thing. The same inconsistency existed in department names.

Duplicates were everywhere due to manual data entry. Extra spaces, mixed data types, partially filled rows, and formatting issues were common.

Because of all this, you couldn’t join anything reliably. There was no safe way to say, “this production record belongs to this worker in the payroll file,” without extensive cleanup, restructuring, and validation first.
 

## The operational side was even worse

Beyond the messy data, there were bigger structural problems:

Some production processes happening on the shop floor weren't even in the ERP. Workers were doing the work, supervisors knew about it, but there was no system record. Just... missing.

You couldn't track how diamonds actually moved through production because the data was too inconsistent. Process names varied, department assignments were unclear, and some steps just weren't captured.

Basic operational reports—like tracking complete production flow by worker or seeing department-level throughput—couldn't be generated. The system either didn't have the reports built in, or they were there but not enabled.

And here's the kicker: the company had expanded from natural diamonds to lab-grown diamonds, but nobody had updated the ERP configuration to reflect this. The system was still set up for the old business model.

## Where I started: making the operational data usable

Before I could analyze anything, I needed to create some structure.

### Consolidating and standardizing operational data

I pulled all the production reports and started building a master list of what actually existed: all the processes, all the departments, all the different variations of how people were naming things.

Then I standardized everything. Every variation of "polishing" became "Polishing." Every department name got normalized to one consistent format. I removed obvious duplicates where the same production entry appeared multiple times.

The goal was simple: make it possible to actually track how diamonds moved through the facility and what happened at each stage. Right now, you couldn't even do that basic tracking because the data was too inconsistent.

### The worker name problem

This was the hard part. Names were so inconsistent that I couldn't just use fuzzy matching and call it done.

I tried RapidFuzz and FuzzyWuzzy first—these are Python libraries that match similar strings even when they're not exact. They work great for small typos and variations.

But my data was beyond that. "Ramesh Kumar" vs "R Kumar"? Fuzzy matching gives that maybe 40% similarity—not enough to confidently say it's the same person. Add in name order reversals, nicknames, and multiple workers with similar names, and automated matching alone wasn't going to cut it.

So I did a hybrid approach:

1. Ran fuzzy matching to catch the obvious ones (90%+ similarity scores)
2. Exported the medium confidence matches (70-89%) for manual review
3. For the rest, I manually matched people by cross-referencing other data:
   - What department they worked in
   - What processes they typically did
   - When they were active (date ranges)
   - The CTS ranges they usually handled

I built a manual mapping table in Excel: messy name → standardized name. Then applied this mapping across all the datasets.

Was it tedious? Extremely. But it was the only way to create a reliable dataset where I could connect someone's production records to their payroll records with confidence.

### Working with the ERP vendor

While I was cleaning data, I also documented what was actually missing from the system. Which processes existed in real life but not in the ERP? Which reports did we need but couldn't generate?

I worked with the software vendor to:
- Add the missing production processes to the system configuration
- Turn on operational reports that existed but weren't enabled  
- Create some new reports for tracking production flow

This wasn't just about my immediate analysis. I was trying to make future data better so the next person wouldn't have to deal with the same mess.

## Understanding how production actually worked

To make sense of all this operational data, I needed to understand the physical production process, not just what the ERP claimed was happening.

I worked on mapping the complete diamond journey—department by department, process by process. This wasn’t a simple or theoretical exercise. The production flow was complex, with multiple handoffs, parallel processes, and CTS values being captured at different stages depending on the type of work.

To do this properly, I created a detailed end-to-end flowchart in Figma. Building it required multiple walkthroughs with department managers and supervisors. I sat with them to understand:

How diamonds actually move from one department to another

What processes happen at each stage

Where CTS measurements are taken and recorded

What a full production cycle looks like in practice, not just on paper

Many of these details were not documented anywhere and, in some cases, differed from how the ERP was configured. The flowchart went through several iterations as I validated assumptions, corrected misunderstandings, and reconciled differences between departments.

Once finalized, this process map became my reference point for everything else.

When I encountered unusual data, I could sanity-check it against reality:

Does this worker performing this process make sense for their department?

Should CTS be captured at this stage of the workflow?

Is this sequence of processes actually possible?

This operational understanding also became critical later during payroll validation. By clearly seeing where CTS should be measured in the production flow, I was able to identify that the ERP was using the wrong CTS reference for certain calculations—specifically, using Making CTS instead of Issue CTS for lab-grown diamonds.

Without building and validating this process flow, that logic error would have been very difficult to detect.

## The payroll validation

Once I had clean operational data, I could tackle the actual payroll question.

Here's what I decided to do differently: instead of just auditing what the ERP calculated, I was going to rebuild the entire payroll calculation myself from scratch.

### How I did it:

I merged the cleaned production data (how many pieces each worker processed, at what CTS weight) with the rate card data (what they should be paid per piece based on CTS ranges and process types).

Then I wrote SQL queries to pull the right data and Python scripts to calculate what each worker should have been paid based on the business rules. Just straight calculation: if you processed X pieces at Y CTS doing Z process, here's what you should get paid.

Then I compared my numbers to what the ERP actually paid them.

### What I found initially:

For most workers, the numbers were pretty close. Small differences that made sense—rounding, timing differences, data quality issues.

But some workers had consistent, significant gaps between what I calculated and what they were paid. Not random variances. Systematic differences.

That's when I started digging into the rate card logic itself.

## The big finding

The company used to work exclusively with natural diamonds. For natural diamonds, the rate calculation uses "Making CTS"—basically the carat weight after the diamond has been processed.

When they expanded into lab-grown diamonds, the business logic should have changed. Lab-grown diamond rates are supposed to be calculated using "Issue CTS"—the carat weight when the diamond is issued to the worker, before they process it.

The ERP was still using Making CTS for everything. Nobody had updated the configuration when the business model changed.

This mattered because Making CTS and Issue CTS are different numbers. Using the wrong one meant workers were getting paid based on the wrong weight measurements.

### The financial impact:

I ran the numbers for a typical month. The company was losing five figures monthly from this one configuration error. Every single month. Ongoing.

And here's the crazy part: this wasn't visible anywhere in the standard ERP reports. The system was generating payroll, producing reports that looked totally normal. Nobody questioned it because the outputs looked reasonable.

The issue only showed up because I independently recalculated everything from the production data, using the process flow I'd mapped to understand which CTS measurement actually mattered.

### Other stuff I caught:

- Rate cards that were missing certain process-CTS combinations entirely
- Cases where the wrong rate got applied because of the messy process names (fuzzy matches accidentally linking to the wrong rate)
- Production records that didn't match what payroll was using, which traced back to the worker name reconciliation issues

## What actually got fixed

### On the operational side:

The company now had standardized operational data they could actually analyze. Process names were consistent. Department tracking was reliable. 

I'd worked with the vendor to add the missing processes to the system, so future production would be captured more completely.

They had operational reports they could actually run to see production flow, track throughput, monitor department performance—things that should have been basic but weren't possible before.

### On the payroll side:

They identified the Making CTS vs Issue CTS configuration error and could fix it. That stopped the recurring monthly loss.

Payroll calculations became more accurate overall because the underlying operational data was cleaner and the rate card logic was corrected.

### System improvements:

The ERP itself was in better shape. Missing processes added. Reports enabled. Better alignment between how work actually happened and how the system captured it.

More importantly, they had an audit-ready dataset structure I'd built that could be used for future analysis without having to redo all the cleanup work.

## What I learned from this

### Operational data quality directly affects everything else

The payroll errors weren't just about rate cards. They came from messy operational data—inconsistent naming, missing records, unclear production tracking. You can't validate financial calculations if you can't reliably figure out what work actually happened.

### Automated matching only gets you partway there

RapidFuzz and FuzzyWuzzy are useful tools, but real-world data is messier than algorithms handle well. The manual validation work—cross-referencing departments, processes, dates to confirm worker identities—was boring and time-consuming but absolutely necessary.

### You have to understand the actual operations

Mapping the physical production flow wasn't just documentation. It drove every analytical decision I made. I needed to know which processes should appear in the data, where measurements should be captured, what a complete cycle looked like, why Making CTS vs Issue CTS mattered. Without that operational understanding, I would have been flying blind.

### Systems don't automatically update when businesses change

The company shifted from natural to lab-grown diamonds. The ERP didn't magically know to change its logic. Assuming the system reflects current reality is dangerous. You have to validate the logic itself, not just check the outputs.

### Sometimes you need to fix the source, not just work around it

I could have built a ton of workarounds for the messy data and kept going. Instead, I worked with the ERP vendor to actually fix things—add missing processes, enable reports, improve configurations. That made future work easier for everyone, not just my immediate project.

### Rebuilding logic reveals systemic problems

If I'd just reconciled payroll totals, I would have seen variances and maybe flagged them. Rebuilding the entire calculation from scratch is what revealed why those variances existed. The Making CTS issue only became clear because I independently calculated what workers should be paid.

### What I should have done differently:

I wasted time early on assuming the ERP logic was fundamentally correct and just looking for data errors. If I'd questioned the system logic itself earlier—specifically asking "wait, are we still using natural diamond logic for lab-grown diamonds?"—I would have caught the issue faster.

## The technical side

### Tools I used:
- Python for most of the heavy lifting (pandas for data manipulation, RapidFuzz and FuzzyWuzzy for name matching, numpy for calculations)
- SQL for pulling and transforming data
- Excel for the manual validation work and building mapping tables
- Jupyter notebooks so I could document what I was doing as I went

### Data scale:
- Thousands of worker records
- Daily production entries going back months
- Multiple departments and dozens of unique process types
- Large datasets that wouldn't work well in just Excel

### My actual workflow:
1. Extract all the ERP reports
2. Run fuzzy matching on names and processes
3. Manually validate the uncertain matches
4. Build standardized mapping tables
5. Apply mappings and clean the datasets
6. Map out the production process flow
7. Validate whether data matched the actual flow
8. Independently recalculate payroll from production data
9. Compare to ERP payroll and analyze variances
10. Dig into the systematic differences to find root causes

## Why this matters for BDA work

This project wasn't sexy. No dashboard, no machine learning, no fancy visualization. Just messy data, unclear requirements, operational complexity, and real business problems.

But this is what a lot of actual business data analysis looks like:

You get handed messy ERP exports and told "figure out if this is right." You have to make the data usable first. You have to understand how the business actually operates before you can analyze anything. You need to know when automation works and when you need to manually validate. You have to question system outputs instead of assuming they're correct. And you need to connect your findings to real business impact.

I showed I could:
- Work with genuinely messy real-world data and make it analysis-ready
- Understand operational processes well enough to validate whether data made sense
- Build independent logic to verify system calculations instead of just auditing outputs
- Work with vendors and stakeholders to fix root causes
- Quantify financial impact from data findings
- Improve systems for future analysis, not just solve immediate problems

That's the kind of work BDAs actually do, especially in companies with legacy systems and complex operations.
