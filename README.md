<h3 align="left">Connect with me:</h3>
<p align="left">
<a href="https://linkedin.com/in/borack" target="blank"><img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/linked-in-alt.svg" alt="borack" height="30" width="40" /></a>
<a href="https://kaggle.com/strixthewatcher" target="blank"><img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/kaggle.svg" alt="strixthewatcher" height="30" width="40" /></a>
</p>

<h3 align="left">Languages and Tools:</h3>
<p align="left"> <a href="https://pandas.pydata.org/" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/2ae2a900d2f041da66e950e4d48052658d850630/icons/pandas/pandas-original.svg" alt="pandas" width="40" height="40"/> </a> <a href="https://www.python.org" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/python/python-original.svg" alt="python" width="40" height="40"/> </a> <a href="https://pytorch.org/" target="_blank" rel="noreferrer"> <img src="https://www.vectorlogo.zone/logos/pytorch/pytorch-icon.svg" alt="pytorch" width="40" height="40"/> </a> <a href="https://www.selenium.dev" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/detain/svg-logos/780f25886640cef088af994181646db2f6b1a3f8/svg/selenium-logo.svg" alt="selenium" width="40" height="40"/> </a> <a href="https://www.sqlite.org/" target="_blank" rel="noreferrer"> <img src="https://www.vectorlogo.zone/logos/sqlite/sqlite-icon.svg" alt="sqlite" width="40" height="40"/> </a> <a href="https://www.tensorflow.org" target="_blank" rel="noreferrer"> <img src="https://www.vectorlogo.zone/logos/tensorflow/tensorflow-icon.svg" alt="tensorflow" width="40" height="40"/> </a> </p>

<h3 align="left">Overview</h3>
If you’ve found your way here, chances are you’ve come from my resume or LinkedIn profile and are interested in reviewing some of my code. This page will discuss my personal projects in reverse chronological order. Each section starts with a TLDR, so you can skim through it if you prefer. The code in the GitHub repositories is just samples from each project offered to showcase my skills as a self-taught developer. It has been altered to remove identifiable characteristics about what websites I’m scraping because I still have the code running and don’t want to invite too much competition. Thanks for stopping by, and please reach out if you have questions! 

<h3 align="left">Kaggle Vesuvius Competition</h3>
<p align="left"><b>First project using: </b>Pytorch, Convolutional Neural Networks, GPUs</p>
<p align="left"><b>TLDR: </b>In a competition to build an AI algorithm capable of detecting the presence of ink on fragments of papyrus charred by the eruption of Mt. Vesuvius, I placed in the top 46% of entrants.</p>

<p><img src="https://raw.githubusercontent.com/jab2727/jab2727/main/DecodingVesuvius.png" alt="Midjourney's interpretation of: analyzing a papyrus scroll damaged in a volcanic eruption using artificial intelligence to look at pixels" style="float:right;width:400px;height:400px;">The most recent project I worked on was a competition on kaggle.com, a well-known website for teaching machine learning and data science. They occasionally host competitions with prize money (in this case, $100k of prizes to those in the top ten), attracting large numbers of data scientists worldwide. The objective of the Vesuvius competition was to detect ink on fragments of papyrus from library scrolls that were all but destroyed in a volcanic eruption centuries ago. </p>

The competition host included some sample code to get us started. They built a very simple Convolutional Neural Network (CNN), a common technique for computer vision problems. If you wrote an AI algorithm that separates cat pictures from dog pictures, a CNN would be the right choice. It “looks” at an image to identify features and then processes those features through convolutional layers to understand what’s in the picture.

My first stab at this problem didn’t include a CNN at all. Looking for ink on a scroll seems like a vision problem, but we’re not trying to see the letter or identify what letter it is. We just want to know if a specific pixel does or does not contain ink. This sounds like a classification problem, potentially much easier to solve. 

With this in mind, I built a model using LightGBM, a technology similar to neural networks in that you train the model on known data and then test its performance on an unseen validation set. Typically it outperforms neural networks on classification and regression problems in speed and accuracy. It only took me a few days to build and test this on the provided data. Unfortunately, it didn’t work. It appears that the feature-finding capability of CNNs is necessary for this problem. I quickly moved on.

For my second attempt, I tried using TensorFlow, the machine-learning Python package I’m most familiar with. This attempt also failed, primarily because of memory constraints. This code has to run in a Kaggle notebook, which offers 28 gigs of RAM on a CPU or a GPU/CPU combo with two banks of 14 gigs. The GPU was about 60x faster; some code took hours to run even with that speed. The CPU was not a viable option. Although I was more comfortable with TensorFlow, PyTorch has a handy dataloader function that significantly reduces the amount of RAM needed to load input data. Pytorch also makes it easier to control what data is loaded to the GPU and CPU, so I can divide it up as evenly as possible and avoid Out Of Memory issues. TensorFlow has a similar memory allocation capability, but I didn’t know about that until later.

Building the first Pytorch model was easy, mainly because some example code was offered. It was slow, but making it 10x faster was no problem. I just had it predict on a grid of 3x3 pixels instead of one pixel at a time. With a functional Pytorch model up and running and producing good results, I began tuning. The vanilla model seemed very prone to gradient explosions. The loss function would decrease to a plateau and then suddenly pop negative. Ending the training before the gradients exploded produced a viable model, but I could see on the leaderboard that other competitors were squeezing much more out of their models.

Using an “ensemble” is a common technique to improve results. The idea is to create a few different models and combine the results to improve accuracy. Because I was limited in how “deep” I could go with one model, I thought perhaps an ensemble would compensate for the lack of depth by adding additional breadth. Ensembles only work if the models are different, so I tried a) varying the number of pixels it looks at and b) varying the layers of the papyrus scroll it looks at (a single sheet of parchment is scanned at 64 different depth layers, and the max we can look at with a model is ten due to RAM constraints, so it’s essential to pick the correct ones). 

When it came time to implement the concept, the solution I came up with was something I’m calling a “vertical ensemble.” I haven’t seen this done anywhere else, but this architecture let me add breadth and depth to my model without causing a gradient explosion. 

First, I train a model as far as possible without exploding, then run the model to make predictions on each full training image. Then I train my second model, but I include the prediction output from the first model as an additional input layer to the second. I train as far as I can again and then make predictions on the images with the second model. For the third model, I include the output from both the first and second models as an input layer into the third model, which makes the final prediction. By structuring my training this way, I could reduce the loss well below where I could get on an individual model. I’m unsure how this might have compared to a traditional ensemble where we just run three models in parallel and average the results. I never tested that because it worked so well on the first try, so I decided my limited resources (access to 30 GPU hours per week) were better spent developing/refining this concept further.

A few other concepts that improved my scores include the following:

1. Letting the model automatically determine which layers to look at. Someone on the message boards shared their research on which layers have the highest signal-to-noise ratio. The conversation focused on which layers were most essential to include on average, but what I found interesting is that different layers were important to different fragments. All fragments are not created equal. It’s possible that the final fragments our models would have to predict on would be significantly different from the three we were given to train on. In my own experimentation, I found that when trying to train on various groupings of layers, the loss rates would drop faster when I selected the most important layers identified in the message board. I also found that when evaluating on unseen test fragments, the standard deviation of predictions generated would be highest when the most important layers were used (which makes sense because when the algorithm has good signal-to-noise, there will be more predictions closer to zero or one, and fewer predictions around 0.5). I used this to ensure I looked at the correct layers when generating predictions on unseen samples.

2. Letting the model dynamically determine its threshold for classifying a pixel based on what percentage of inked pixels a certain threshold would yield. The model predicts the probability that a pixel is ink, a value from 0 to 1. For example, a 0.24 would indicate a 24% chance a pixel has ink. The file my algorithm produces must be zeroes or ones, not probabilities. To convert probabilities to zeroes and ones, we specify a threshold value. A good starting point might be 0.50, but if my model has higher confidence on a training set than on a validation or test set, that confidence might not exceed the threshold needed to label a pixel as inked. My early test submissions produced poor results because I assumed a threshold of 0.50 would be common sense. Reducing my threshold to somewhere in the 0.20-0.30 range significantly improved my results, indicating that my model was less confident on the test data. From looking at the ground-truth files, I know that somewhere in the range of 10-12% of fragment pixels contain ink, so instead of setting a hard threshold, I built an algorithm to slowly increase the threshold from .01 to .99 by increments of .01 until 12% of pixels are labeled as containing ink. Instead of asking my algorithm “Which pixels are you greater than x% confident contain ink”, I’m asking it, “Which 12% of pixels are you most confident contain ink”. 

By the end of the competition, I placed in the top 46%. I expected to do better, but I’m satisfied with being in the top half for my first competition. My code is broken into three notebooks. Separating the code this way helped manage memory issues. 

Notebook 1: **https://www.kaggle.com/code/jeffborack/vesuvius-v4-1-train**
This notebook shows how I trained the Pytorch model.

Notebook 2: **https://www.kaggle.com/code/jeffborack/vesuvius-v6-1-eval**
This notebook was used to make predictions on the training files, which were saved and added as layers to the second and third models.

Notebook 3: **https://www.kaggle.com/code/jeffborack/vesuvius-6-1-submit**
This is the code that was submitted to the competition. It takes an input file, works through three prediction layers, and assembles a run-line-encoding output file.

<h3 align="left">Sportsbetting AI</h3>
<p align="left"><strong>First project using: </strong>SQL, LightGBM, Numpy</p>
<p align="left"><strong>Repository: </strong></p>
<p align="left"><strong>TLDR: </strong>I lost $3 betting on sports with an AI tool I built, but I hope to do better next season!</p>

My second big Python project was a TensorFlow/LightGBM AI model to bet on fantasy basketball. I’m not a big sports fan, and I’ve probably watched 3 NBA games my entire life. I don’t even know the positions. I simply wanted to scrape data, plug it into a model, and see if it could win bets.

I’ve always been interested in sports betting. In the early 2000’s I was able to make some money betting on UFC fights. The UFC was popular enough that sportsbooks had lines but small enough that occasionally unknown fighters walked into championship fights. Crazy. When that happened, I would do the legwork of finding out where they train, calling the gym or the trainer, and asking questions about the fighter’s background and experience. Once, I managed to get pictures of a fighter nobody else had seen. It was a fun time, but it didn’t last long.

In the 2010s, when sportsbooks introduced live betting, I stumbled across a book about building financial-like sports betting models in Excel. Hockey is a low-scoring game where things can change in an instant, and I suspected that the market would be overreacting to one-goal leads at the end of the first period, especially for underdogs. I spent some time manually recording betting lines during each period. After about a season of recording data, I concluded that my instincts were wrong. I couldn’t identify any consistently exploitable inefficiencies. 

That brings us to 2022. I started this project around September and hoped to finish it before January, but I didn’t have a functional model until April. My first goal was to learn how to use SQL, and the first step in the project was scraping massive amounts of data. Instead of using NumPy arrays and Pandas to save CSV files, I saved all the data I was scraping to SQL databases. This gave me confidence that my data was being stored safely, making it much faster to access and modify. 

My second goal was to build a reinforcement learning model similar to AlphaGo. The model would compete against itself to draft the best fantasy basketball team it could for each night of games. With access to large amounts of data stretching back to 2010, it would repeatedly play until it could draft the ultimate fantasy team. My attempts to build such a model failed for two key reasons, a) size, and b) complexity. 

Size: Up to 14 NBA games could occur in a night, which equals 28 teams, with 18 players each plus a coach. Leveraging the database, I constructed a snapshot of each player's last 30 games, with about 25 relevant stats per player. That’s roughly 400k data points for every single game night. My computer couldn’t run this. Google Cloud wouldn’t allow me to purchase time on computers with enough ram to run this. AWS granted me permission to access the necessary resources, but it cost me about $90/hour to work on. Those hours added up quickly and put pressure on me to change course.

Complexity: Reinforcement learning is the pinnacle of AI. Even without a strong background in coding, my early attempts at building neural networks with TensorFlow were very successful. I got models up and running quickly, producing realistic results. Reinforcement learning is much more difficult because the function you use to calculate loss must be differentiable to properly update the gradients between neurons (or nodes). The formula that calculates the agent’s score at the end of each round is not differentiable. There are tricks to make it differentiable, but my understanding of the underlying math was insufficient to get me over that hurdle. If not for the size issue mentioned above, I would have been more comfortable spending time on it, but compute time was too expensive. 

After a few months of failing (and watching the NBA season slip by), I returned to the drawing board and asked myself what a minimally viable product looked like for a sports betting AI. I re-engineered the project so that instead of drafting a team, the Tensorflow model would estimate how many fantasy points a player would earn each night. This simplified my model and reduced my dataset size by 93% (by allowing me to look at each matchup rather than a whole night of games). A player’s projected performance could be used to construct my own fantasy teams. Not ideal, but better than nothing. With a more reasonable goal set, it took about two weeks to have a working model.

The next day I put $100 into a DraftKings account. I started betting based on my model’s predictions, and from that first night, the model made money. Despite the success, I dug into the projections, trying to figure out where I could improve by tracking where my model’s predictions were divergent from those offered on other sports betting websites. I also tracked the mean squared error produced by my model vs. others. This allowed me to identify some issues and steadily improve the algorithm and how my databases are organized. 

Heading into the playoffs, my account was up about 30%. That’s only $30, but as a proof of concept, I was pleased. After all, the odds in sports betting are against you, and I knew nothing about basketball. However, in the last two weeks of the season, I lost back the most challenging $30 I had ever earned. It wasn’t totally the model’s fault. I would have prepared for this if I knew more about sports. In the last few weeks of the season, the teams that have secured a spot in the playoffs rest their best players to prevent/recover from injuries. The teams that know they won’t make it to the playoffs also rest their best players to give some of the junior players (or g-leaguers) a try. My model failed to predict that teams would suddenly stop playing their best players, so all the predictions were wrong. 

Next year I will try rerunning the model to see how it performs. There are a few other incremental improvements to make before then, and I’ll be prepared to call it quits a few weeks before playoffs.

<h3 align="left">Collectibles Trading Project</h3>
<p align="left"><strong>First project using: </strong>Python, Selenium, BeautifulSoup, TensorFlow, Pandas</p>
<p align="left"><strong>Repository: </strong>**https://github.com/jab2727/CollectibleTrading**</p>
<p align="left"><strong>TLDR: </strong>I built an app that scrapes websites and uses Tensorflow to automatically buy and sell mispriced items.</p>

My first actual coding project was a tool to buy and sell collectibles online. The repo contains two files. The first is Price_Scraper, which scans through pages of products trying to identify items to purchase. When I started the project, I tried using Beautiful Soup for web scraping because I had some familiarity with it. However, it didn’t work. There was some data it just couldn’t detect. I got to the point where I considered quitting, but my curiosity about what I was doing wrong wouldn’t let me give up. As a last-ditch effort, I hired a developer on Fiverr to help me get the code working. It took him about 10 minutes to install Selenium, and he was able to scrape all the data I couldn’t get with Beautiful Soup. The critical difference between Selenium and Beautiful Soup is that Selenium handles dynamic content that loads over time.

With Selenium running, I could scrape all the data I needed from each page, enter data into text fields, and interact with buttons and dropdown menus. The next step was to identify good items to buy. The website I scrape provides a price chart for each item. I built a Tensorflow model to “look” at the price chart and determine if the price is stable, increasing, or decreasing. If the price is falling, I ignore it. If the price is stable or rising, we add it to a list of items to have another algorithm look at later. The score it receives is saved so that we can bid more aggressively on increasing value items.

The code where I built the TensorFlow model isn’t included, but that was probably the easiest part of the project. I manually looked at about 2,000 price charts to create the dataset and recorded whether I thought it was good, bad, or neutral. After that, I supervised the algorithm as it started to make predictions. If it was confident in its own prediction, I accepted it. If the confidence was low, I would have the algorithm stop momentarily and ring a bell to get my attention. I’d enter the correct score, save it to my training data set, and the algorithm would continue. Over a few months, I retrained the model perhaps once per week, and it improved its confidence. With about 8,000 examples, the model is now excellent at predicting how I would grade a price chart, and the scanning process is now completely automated.

The Daily_Routine is the main file in this project. It runs continuously, spending about 8 hours daily updating prices on items I’m trying to buy or sell and scanning the list of new items with attractive price charts to bid on. After working through its loop 5 times, it takes a break until the following morning. I only relaunch it when I have Chrome updates to install. The critical point of showcasing this code is that there was some decent complexity to it. It’s a little over 700 lines, with numerous defined functions being called. Given that it’s my first actual coding project, in hindsight, it’s a bit disorganized, and I did a few things incorrectly/inefficiently, but it works, and I’m proud of it.

<h3 align="left">Coding Background Prior to Summer 2022</h3>
I first tried learning to code in 1999. Inspired by The Matrix, I went to Barnes & Noble to buy a book on coding. I left with a book on Visual Basic because the name indicated it would be both visual and basic. Fantastic! I learned a lot about logic loops and syntax, and it felt great to get my computer to do things, but I realized that Visual Basic was probably not the language of choice for Neo and Trinity. It also couldn’t make fun computer games. It was designed for building incredibly boring business apps. After finishing the book, I didn’t work on any projects.

In college, I majored in bioengineering. C++ was a prereq for everyone in engineering, and my final project was a respectable calculator. The bioengineering program was way ahead of its time. We studied neural networks and used a language called Matlab to try to build some functional prototypes. In 2004, neural networks weren’t yet producing impressive results, but the concept was fascinating, and I devoured books on the subject. Synaptic Self, The Singularity Is Near, Gödel Escher Bach, and several other books by Douglas Hofstadter and Greg Egan. I kept a loose watch on the space as I graduated college and started my career in finance. 

In 2013 I took another shot at learning to code and completed a Codecademy class on Python. From that, I built a simple web scraping tool. If I was smarter, I would have started exploring neural networking again, as IBM Watson was making headlines, but instead, I decided to launch a coffee startup. It wasn’t until last summer that I turned my undivided attention back to learning to code. Perhaps my most significant source of motivation was my 8-year-old son. I had been stressing the importance of learning to code from an early age, and he had worked his way through some Scratch projects, CodeCombat, and code.org. I could tell from his questions that he would soon need more guidance than I could offer without knowing how to code myself. I also felt like a hypocrite telling my kid how important it is to code without knowing how to code myself. With the kid at summer camp, I began working on the collectibles trading app.

- 📫 How to reach me **jborack@gmail.com**
