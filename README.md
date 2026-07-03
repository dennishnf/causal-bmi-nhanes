# Causal analysis of BMI using NHANES data

This repo holds the code and the write up for a small project I did on the NHANES health survey. The goal isn't to predict BMI. It's to ask a harder question, which is what would happen to a person's BMI if they changed how much they move or what they eat. Plain machine learning can't really answer that, so most of the work here is about setting the problem up as a causal one.

I look at four things that might affect BMI. Those are doing vigorous physical activity, and daily intake of total calories, protein, and carbohydrate. The whole thing runs inside one Jupyter notebook.

## What's in here

- `Causal_BMI_NHANES.ipynb` is the main file. It runs the full pipeline from start to finish, from cleaning the data all the way to the final numbers and plots.
- `requirements.txt` has the exact package versions I used.
- `nhanes.csv` is the dataset. You'll need to add this one yourself, and there's a note on that below.
- `Causal_BMI_NHANES.pdf` is the short paper/report that explains the whole thing in words, with the figures.

## The data

The notebook reads a file called `nhanes.csv` from the same folder it sits in. It's a cross sectional slice of NHANES covering a few thousand adults, with demographics, body measures, activity levels, and a one day diet recall for each person.

Just drop the csv in the repo root, next to the notebook, and keep the name as `nhanes.csv`. If your copy lives somewhere else, you can edit the `DATA_PATH` line near the top of the cleaning section and point it wherever you like.

One heads up. The raw file has a couple of strange codings that the notebook fixes on its own. A very tiny float stands in for zero in some columns, and the number 9999 in the sedentary column means the person refused to answer, not that they sat still for a whole week. The cleaning step turns both of those into missing values, caps calories to a sensible adult range, and drops rows with gaps. That leaves around 3700 people out of the original 3842.

## Running it

You'll want Python 3.10. Install the packages first, then open the notebook.

```
pip install -r requirements.txt
jupyter notebook Causal_BMI_NHANES.ipynb
```

From there just run the cells top to bottom. There's one fixed random seed for everything that uses randomness, so you should get the same numbers and the same figures every time you run it.

Small warning, the causal discovery and the bootstrap step take a little while, and the double machine learning part is the slowest bit, so give it a moment.

## What the notebook actually does

It comes in two halves.

The first half learns the causal structure from the data instead of assuming I already know it. I run three discovery algorithms that each work in a different way, PC, GES, and DirectLiNGAM, and I look at where they agree and where they fall apart. Then I resample the rows and rerun one of them a bunch of times to see which edges are stable and which ones come and go. I take the stable edges, mix in a short list of physiology rules that no algorithm is allowed to break, like nothing can cause your age, and build one working graph out of that.

The second half uses that graph to decide what to control for, and then estimates the four effects with confidence intervals. After that I spend most of the effort trying to break the main result, using a placebo test, an invented confounder, a subset check, a sensitivity scan for hidden confounders, and a double machine learning cross check. There's also a piece near the end about the gap between a plain diet effect and an energy held one, which matters a lot for how you read the protein and carbohydrate numbers.

## What comes out

The clearest result is that vigorous activity goes with a BMI about 1.7 points lower after adjustment, and that one holds up under every check I threw at it. The diet effects are small, and they shift around a lot depending on how you frame the question, so I wouldn't read too much into them.

I want to be honest about the ceiling here. The data is a single snapshot per person, so for any of these the arrow could point the other way. Heavier people might move less because of their weight rather than the reverse, and no amount of adjusting fixes a backwards arrow. I also didn't apply the survey weights, so the numbers describe this sample and not the whole country. Read every number as a carefully adjusted association that carries a causal reading, not as proven cause. The paper goes into all of this in more detail.

## The paper

The full write up is `DennisNunez_CausalBMI.pdf` in this repo. It walks through the graphs, the tables, and the checks with all the figures next to them, so if you want the reasoning rather than the code, start there.
