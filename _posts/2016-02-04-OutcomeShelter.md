<!DOCTYPE html>

<html xmlns="http://www.w3.org/1999/xhtml">

<head>

<meta charset="utf-8">
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<meta name="generator" content="pandoc" />
<meta name="viewport" content="width=device-width, initial-scale=1">

<meta name="author" content="Alexandru Papiu" />


<title>Visualizing Breed and Age by Outcome</title>

<script src="__results___files/jquery-1.11.3/jquery.min.js"></script>
<meta name="viewport" content="width=device-width, initial-scale=1" />
<link href="__results___files/bootstrap-3.3.5/css/readable.min.css" rel="stylesheet" />
<script src="__results___files/bootstrap-3.3.5/js/bootstrap.min.js"></script>
<script src="__results___files/bootstrap-3.3.5/shim/html5shiv.min.js"></script>
<script src="__results___files/bootstrap-3.3.5/shim/respond.min.js"></script>


<style type="text/css">code{white-space: pre;}</style>
<style type="text/css">
div.sourceCode { overflow-x: auto; }
table.sourceCode, tr.sourceCode, td.lineNumbers, td.sourceCode {
  margin: 0; padding: 0; vertical-align: baseline; border: none; }
table.sourceCode { width: 100%; line-height: 100%; background-color: #f8f8f8; }
td.lineNumbers { text-align: right; padding-right: 4px; padding-left: 4px; color: #aaaaaa; border-right: 1px solid #aaaaaa; }
td.sourceCode { padding-left: 5px; }
pre, code { background-color: #f8f8f8; }
code > span.kw { color: #204a87; font-weight: bold; } /* Keyword */
code > span.dt { color: #204a87; } /* DataType */
code > span.dv { color: #0000cf; } /* DecVal */
code > span.bn { color: #0000cf; } /* BaseN */
code > span.fl { color: #0000cf; } /* Float */
code > span.ch { color: #4e9a06; } /* Char */
code > span.st { color: #4e9a06; } /* String */
code > span.co { color: #8f5902; font-style: italic; } /* Comment */
code > span.ot { color: #8f5902; } /* Other */
code > span.al { color: #ef2929; } /* Alert */
code > span.fu { color: #000000; } /* Function */
code > span.er { color: #a40000; font-weight: bold; } /* Error */
code > span.wa { color: #8f5902; font-weight: bold; font-style: italic; } /* Warning */
code > span.cn { color: #000000; } /* Constant */
code > span.sc { color: #000000; } /* SpecialChar */
code > span.vs { color: #4e9a06; } /* VerbatimString */
code > span.ss { color: #4e9a06; } /* SpecialString */
code > span.im { } /* Import */
code > span.va { color: #000000; } /* Variable */
code > span.cf { color: #204a87; font-weight: bold; } /* ControlFlow */
code > span.op { color: #ce5c00; font-weight: bold; } /* Operator */
code > span.pp { color: #8f5902; font-style: italic; } /* Preprocessor */
code > span.ex { } /* Extension */
code > span.at { color: #c4a000; } /* Attribute */
code > span.do { color: #8f5902; font-weight: bold; font-style: italic; } /* Documentation */
code > span.an { color: #8f5902; font-weight: bold; font-style: italic; } /* Annotation */
code > span.cv { color: #8f5902; font-weight: bold; font-style: italic; } /* CommentVar */
code > span.in { color: #8f5902; font-weight: bold; font-style: italic; } /* Information */
</style>
<style type="text/css">
  pre:not([class]) {
    background-color: white;
  }
</style>



<base target="_blank" /></head>

<body>
                                        <script>
function postHeightMessage() {
  var mainDiv = document.getElementById('notebook');
  if(mainDiv===null){
    mainDiv = document.getElementsByClassName('main-container')[0];
  }
  parent.postMessage(mainDiv.scrollHeight, "*");
}
document.addEventListener("DOMContentLoaded", postHeightMessage );
window.onload = function() { postHeightMessage;
    window.setTimeout(postHeightMessage, 500);
    window.setTimeout(postHeightMessage, 1500);
}
</script>

<style type = "text/css">
.main-container {
  max-width: 940px;
  margin-left: auto;
  margin-right: auto;
}
code {
  color: inherit;
  background-color: rgba(0, 0, 0, 0.04);
}
img {
  max-width:100%;
  height: auto;
}
h1 {
  font-size: 34px;
}
h1.title {
  font-size: 38px;
}
h2 {
  font-size: 30px;
}
h3 {
  font-size: 24px;
}
h4 {
  font-size: 18px;
}
h5 {
  font-size: 16px;
}
h6 {
  font-size: 12px;
}
.tabbed-pane {
  padding-top: 12px;
}
button.code-folding-btn:focus {
  outline: none;
}
</style>


<div class="container-fluid main-container">

<!-- tabsets -->
<script src="__results___files/navigation-1.0/tabsets.js"></script>
<script>
$(document).ready(function () {
  window.buildTabsets("TOC");
});
</script>

<!-- code folding -->






<div class="fluid-row" id="header">


<h1 class="title">Visualizing Breed and Age by Outcome</h1>
<h4 class="author"><em>Alexandru Papiu</em></h4>

</div>


<p>There have already been a lot of really great scripts visualizing this data so what I’d like to do is focus on a few aspects of the data that I think might be important but were a little trickier to visualize. The four aspects I will focus on are age, breed, color and time of week/day.</p>
<p>Let’s load in the required packages and data for starters:</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class="kw">library</span>(dplyr)
<span class="kw">library</span>(ggplot2)
<span class="kw">library</span>(gridExtra)
<span class="kw">library</span>(lubridate)
train &lt;-<span class="st"> </span><span class="kw">read.csv</span>(<span class="st">&#39;../input/train.csv&#39;</span>, <span class="dt">stringsAsFactors =</span> F)</code></pre></div>
<div id="animal-age-upon-outcome" class="section level2">
<h2>Animal Age upon Outcome:</h2>
<p>First I’d like find a good way to see how the age of the animal influences the shelter outcome. Age is usually pretty straightforward to deal with but in this case it comes in a slightly odd format. Here are the first five entries:<code>1 year, 1 year, 2 years, 3 weeks, 2 years</code></p>
<p>The issue here is that there are too many factors and they are not in order so what I will do is group some factors together and then reorder them. Maybe there is a better way to do this but I pretty much do it manually, joining certain groups together.</p>
<p>One we have reordered we are in a good position to visualize the data using stacked bar charts with age as the x variable. While the stacked relative frequency graphs are great at showing what outcomes are more likely for different ages they do not show the actual count of animals for given age groups. To remedy this I am adding a histogram on top of the relative frequency of outcomes. This way we can have a more complete picture of how outcome is influenced by age.</p>
<p>Also, per Andy’s suggestions in the comments I am doing two plots - one for dogs and one for cats. Interestingly there are quite a few differences by type of animals. Cats, for example, see a decrease in adoption rate from 2 months to 2 years but then surisingly the adoption rate inces up a bit for older cats.</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r">cat &lt;-<span class="st"> </span><span class="kw">filter</span>(train, AnimalType ==<span class="st"> &quot;Cat&quot;</span>)

cat %&gt;%<span class="st"> </span><span class="kw">count</span>(age) %&gt;%<span class="st">  </span>
<span class="st">    </span><span class="kw">ggplot</span>(<span class="kw">aes</span>(<span class="dt">x =</span> age, <span class="dt">y =</span> n)) +
<span class="st">        </span><span class="kw">geom_bar</span>(<span class="dt">stat =</span> <span class="st">&quot;identity&quot;</span>, <span class="dt">alpha =</span> <span class="fl">0.5</span>) +
<span class="st">        </span><span class="kw">theme_void</span>() +<span class="st"> </span>
<span class="st">        </span><span class="kw">ylab</span>(<span class="st">&quot;Count&quot;</span>) +<span class="st"> </span>
<span class="st">        </span><span class="kw">ggtitle</span>(<span class="st">&quot;Outcome Type by Age upon Outcome - Cats&quot;</span>) +
<span class="st">        </span><span class="kw">theme</span>(<span class="dt">axis.title.y =</span> <span class="kw">element_text</span>(<span class="dt">angle =</span> <span class="dv">90</span>, <span class="dt">color =</span> <span class="st">&quot;#737373&quot;</span> )) -&gt;<span class="st"> </span>g1

<span class="kw">ggplot</span>(cat, <span class="kw">aes</span>(<span class="dt">x =</span> age, <span class="dt">fill =</span> OutcomeType)) +
<span class="st">     </span><span class="kw">geom_bar</span>(<span class="dt">stat =</span> <span class="st">&quot;count&quot;</span>, <span class="dt">position =</span> <span class="st">&quot;fill&quot;</span>,  <span class="dt">alpha =</span> <span class="fl">0.9</span>) +
<span class="st">     </span><span class="kw">scale_fill_brewer</span>(<span class="dt">palette =</span> <span class="st">&quot;Set1&quot;</span>) +
<span class="st">     </span><span class="kw">theme_void</span>() +<span class="st"> </span>
<span class="st">     </span><span class="kw">ylab</span>(<span class="st">&quot;Relative Count by Outcome&quot;</span>) +
<span class="st">     </span><span class="kw">theme</span>(<span class="dt">axis.text.x =</span> <span class="kw">element_text</span>(<span class="dt">angle =</span> <span class="dv">90</span>, <span class="dt">vjust =</span> <span class="fl">0.5</span>, <span class="dt">size =</span> <span class="dv">11</span>, <span class="dt">color =</span> <span class="st">&quot;#737373&quot;</span>),
           <span class="dt">legend.position =</span> <span class="st">&quot;bottom&quot;</span>,
           <span class="dt">axis.title.y =</span> <span class="kw">element_text</span>(<span class="dt">angle =</span> <span class="dv">90</span>, <span class="dt">color =</span> <span class="st">&quot;#737373&quot;</span>)) -&gt;<span class="st"> </span>g2

<span class="kw">grid.arrange</span>(g1, g2, <span class="dt">ncol =</span> <span class="dv">1</span>, <span class="dt">heights =</span> <span class="kw">c</span>(<span class="dv">1</span>,<span class="dv">3</span>))</code></pre></div>
<p><img src="__results___files/figure-html/unnamed-chunk-3-1.png" title="" alt="" width="1152" /></p>
<p><img src="__results___files/figure-html/unnamed-chunk-4-1.png" title="" alt="" width="1152" /></p>
<p>What else do the graphs above tell us? Interestingly animals that are less than one month old are almost always transferred. Keep in mind that we are looking at age upon outcome so a data point represents a certain action being taken - most young animals will probably stay put and not make it to this dataset. Once the animal is not a baby anymore it reaches its peak in terms of adoption probability very early - at around 2 months. From there on the chances of adoption decrease gradually while the chances of both euthanasia and return to the owner increase. Fortunately even old animals are seldomly euthanasiazed.</p>
</div>
<div id="breeds" class="section level2">
<h2>Breeds:</h2>
<p>There are 1380 unique breeds in our dataset. This is way too many to try to visualize factor by factor. We will have to come up with a way to find coarser group for our breeds.</p>
<p>For starters I will borrow an idea from Megan Risdals’s excellent script <a href="https://www.kaggle.com/mrisdal/shelter-animal-outcomes/quick-dirty-randomforest">here</a> and remove the “Mix” word (which would be better used as a separate feature) and only keep the first breed:</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class="kw">gsub</span>(<span class="st">&quot; Mix&quot;</span>, <span class="st">&quot;&quot;</span>, train$Breed) -&gt;<span class="st"> </span>temp

<span class="kw">strsplit</span>(<span class="dt">x =</span> temp, <span class="dt">split =</span> <span class="st">&quot;/&quot;</span>) %&gt;%<span class="st"> </span><span class="kw">sapply</span>(function(x){x[<span class="dv">1</span>]}) -&gt;<span class="st"> </span>train$breed1</code></pre></div>
<p>Even once we do this we still have more than 200 breeds but fortunately the majority of animals fall within the better known breeds - the top 15 breeds cover more than 75% of the dataset.</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r">train %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">count</span>(breed1) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">arrange</span>(<span class="kw">desc</span>(n)) %&gt;%<span class="st">  </span><span class="kw">head</span>(<span class="dv">15</span>) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">ggplot</span>(<span class="kw">aes</span>(<span class="dt">x =</span> <span class="kw">reorder</span>(breed1, n), <span class="dt">y =</span> n)) +
<span class="st">    </span><span class="kw">geom_bar</span>(<span class="dt">stat =</span> <span class="st">&quot;identity&quot;</span>, <span class="dt">width =</span> <span class="fl">0.8</span>) +
<span class="st">    </span><span class="kw">coord_flip</span>() +<span class="st"> </span>
<span class="st">    </span><span class="kw">theme</span>(<span class="dt">axis.title.y =</span> <span class="kw">element_blank</span>()) +
<span class="st">    </span><span class="kw">ggtitle</span>(<span class="st">&quot;Popular Breeds&quot;</span>) +
<span class="st">    </span><span class="kw">ylab</span>(<span class="st">&quot;Number of Animals&quot;</span>)</code></pre></div>
<p><img src="__results___files/figure-html/unnamed-chunk-6-1.png" title="" alt="" width="672" /></p>
<p>What we’ll do at this point is select the breeds that are most popular and identify all other breeds as exotic. Let’s call a breed exotic if there are less than 150 animals in our dataset with the given breed - this will decrease the variance for the types of outcomes and hopefully allow us to see the patterns that are really there. Then we’ll plot the the breeds by outcome. Since cats and dogs tend to have different outcomes we will do two separate plots, one for dogs and one for cats.</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class="kw">count</span>(train, breed1) %&gt;%
<span class="st">    </span><span class="kw">arrange</span>(<span class="kw">desc</span>(n)) %&gt;%
<span class="st">    </span><span class="kw">filter</span>(n &gt;<span class="dv">150</span>) -&gt;<span class="st"> </span>popular

train$breed1[!(train$breed1 %in%<span class="st"> </span>popular$breed1)] &lt;-<span class="st"> &quot;Exotic&quot;</span>


<span class="co">#a look at the plots:</span>
<span class="kw">filter</span>(train, AnimalType ==<span class="st"> &quot;Cat&quot;</span>) %&gt;%<span class="st"> </span>
<span class="kw">ggplot</span>(<span class="kw">aes</span>(<span class="dt">x =</span> breed1, <span class="dt">fill =</span> OutcomeType)) +
<span class="st">    </span><span class="kw">geom_bar</span>(<span class="dt">stat =</span> <span class="st">&quot;count&quot;</span>, <span class="dt">position =</span> <span class="st">&quot;fill&quot;</span>, <span class="dt">width =</span> <span class="fl">0.7</span>) +<span class="st"> </span>
<span class="st">    </span><span class="kw">coord_flip</span>() +
<span class="st">    </span><span class="kw">ggtitle</span>(<span class="st">&quot; Cat Breeds by Outcome&quot;</span>) +<span class="st"> </span>
<span class="st">    </span><span class="kw">scale_fill_brewer</span>(<span class="dt">palette =</span> <span class="st">&quot;Set1&quot;</span>) +
<span class="st">    </span><span class="kw">theme</span>(<span class="dt">axis.title =</span> <span class="kw">element_blank</span>())</code></pre></div>
<p><img src="__results___files/figure-html/unnamed-chunk-7-1.png" title="" alt="" width="672" /></p>
<p>It looks like there is some variation in the outcomes based on cat breeds. It might be more useful if we could order the plot above based on, say, fraction on cats adopted. Let’s do this below using a Cleveland dot graph.</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r">breedz &lt;-<span class="st"> </span><span class="kw">data.frame</span>(<span class="dt">breed =</span> train$breed1, <span class="dt">outcome =</span> train$OutcomeType, <span class="dt">type =</span> train$AnimalType)

breedz %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">group_by</span>(type, breed, outcome) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">summarise</span>(<span class="dt">n =</span> <span class="kw">n</span>()) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">mutate</span>(<span class="dt">frac =</span>  n/<span class="kw">sum</span>(n)) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">ungroup</span>() %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">filter</span>(outcome ==<span class="st"> &quot;Adoption&quot;</span>, type ==<span class="st"> &quot;Cat&quot;</span>) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">arrange</span>(frac) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">ggplot</span>(<span class="kw">aes</span>(<span class="dt">x =</span> <span class="kw">reorder</span>(breed, frac), <span class="dt">y =</span> frac)) +
<span class="st">    </span><span class="kw">geom_point</span>(<span class="dt">size =</span> <span class="dv">3</span>, <span class="dt">color =</span> <span class="st">&quot;dark blue&quot;</span>) +
<span class="st">    </span><span class="kw">theme_minimal</span>() +
<span class="st">    </span><span class="kw">coord_flip</span>() +
<span class="st">    </span><span class="kw">ylab</span>(<span class="st">&quot;Fraction Adopted&quot;</span>) +
<span class="st">    </span><span class="kw">ggtitle</span>(<span class="st">&quot;Adoption Rate by Breed - Cats&quot;</span>) +
<span class="st">    </span><span class="kw">theme</span>(<span class="dt">axis.title.y =</span> <span class="kw">element_blank</span>())</code></pre></div>
<p><img src="__results___files/figure-html/unnamed-chunk-8-1.png" title="" alt="" width="672" /></p>
<p>This looks quite interesting! There is almost a linear trend in fraction of cats adopted based on breed. Not only that but the adoption rate is inversely proportional to how exotic a certain breed is: lesser known breeds tent to be adopted at a higher rate than common breeds like the <code>Domestic Shorthair</code>.</p>
<p>Now let’s take a look at the dogs:</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class="kw">filter</span>(train , AnimalType ==<span class="st"> &quot;Dog&quot;</span>) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">ggplot</span>(<span class="kw">aes</span>(<span class="dt">x =</span> breed1, <span class="dt">fill =</span> OutcomeType)) +
<span class="st">    </span><span class="kw">geom_bar</span>(<span class="dt">stat =</span> <span class="st">&quot;count&quot;</span>, <span class="dt">position =</span> <span class="st">&quot;fill&quot;</span>, <span class="dt">width =</span> <span class="fl">0.8</span>) +<span class="st"> </span>
<span class="st">    </span><span class="kw">ggtitle</span>(<span class="st">&quot;Dog Breeds by Outcome&quot;</span>) +
<span class="st">    </span><span class="kw">coord_flip</span>() +
<span class="st">    </span><span class="kw">scale_fill_brewer</span>(<span class="dt">palette =</span> <span class="st">&quot;Set1&quot;</span>) +
<span class="st">    </span><span class="kw">theme</span>(<span class="dt">axis.title =</span> <span class="kw">element_blank</span>())</code></pre></div>
<p><img src="__results___files/figure-html/unnamed-chunk-9-1.png" title="" alt="" width="672" /></p>
<p>And let’s reorder these based on adoption rates as well:</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r">breedz %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">group_by</span>(type, breed, outcome) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">summarise</span>(<span class="dt">n =</span> <span class="kw">n</span>()) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">mutate</span>(<span class="dt">frac =</span>  n/<span class="kw">sum</span>(n)) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">ungroup</span>() %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">filter</span>(outcome ==<span class="st"> &quot;Adoption&quot;</span>, type ==<span class="st"> &quot;Dog&quot;</span>) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">arrange</span>(frac) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">ggplot</span>(<span class="kw">aes</span>(<span class="dt">x =</span> <span class="kw">reorder</span>(breed, frac), <span class="dt">y =</span> frac)) +
<span class="st">    </span><span class="kw">geom_point</span>(<span class="dt">size =</span> <span class="dv">3</span>, <span class="dt">color =</span> <span class="st">&quot;dark blue&quot;</span>) +
<span class="st">    </span><span class="kw">theme_minimal</span>() +
<span class="st">    </span><span class="kw">coord_flip</span>() +
<span class="st">    </span><span class="kw">ylab</span>(<span class="st">&quot;Fraction Adopted&quot;</span>) +
<span class="st">    </span><span class="kw">ggtitle</span>(<span class="st">&quot;Adoption Rate by Breed - Dogs&quot;</span>) +
<span class="st">    </span><span class="kw">theme</span>(<span class="dt">axis.title.y =</span> <span class="kw">element_blank</span>())</code></pre></div>
<p><img src="__results___files/figure-html/unnamed-chunk-10-1.png" title="" alt="" width="672" /></p>
<p>Shih Tzu seems to be the outlier here: very low adoptions rates - around 10%, but also very low euthanasia rates. Most Shih Tzu’s are transfered or returned. On thing to note is that there aren’t that many dogs of this breed in the data set - only 176. My theory here would be that Shih Tzu’s seem to be a little too fancy of a breed for people to adopt. Most people looking for one would probably go to a breeder so that they have more information about the dog. This would explain the low adoption rates.</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r">breedz %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">group_by</span>(type, breed, outcome) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">summarise</span>(<span class="dt">n =</span> <span class="kw">n</span>()) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">mutate</span>(<span class="dt">frac =</span>  n/<span class="kw">sum</span>(n)) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">ungroup</span>() %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">filter</span>(outcome ==<span class="st"> &quot;Euthanasia&quot;</span>, type ==<span class="st"> &quot;Dog&quot;</span>) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">arrange</span>(frac) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">ggplot</span>(<span class="kw">aes</span>(<span class="dt">x =</span> <span class="kw">reorder</span>(breed, frac), <span class="dt">y =</span> frac)) +
<span class="st">    </span><span class="kw">geom_point</span>(<span class="dt">size =</span> <span class="dv">3</span>, <span class="dt">color =</span> <span class="st">&quot;dark blue&quot;</span>) +
<span class="st">    </span><span class="kw">theme_minimal</span>() +
<span class="st">    </span><span class="kw">coord_flip</span>() +
<span class="st">    </span><span class="kw">ylab</span>(<span class="st">&quot;Fraction Euthanised&quot;</span>) +
<span class="st">    </span><span class="kw">ggtitle</span>(<span class="st">&quot;Euthanasia Rates by Breed - Dogs&quot;</span>) +
<span class="st">    </span><span class="kw">theme</span>(<span class="dt">axis.title.y =</span> <span class="kw">element_blank</span>())</code></pre></div>
<p><img src="__results___files/figure-html/unnamed-chunk-11-1.png" title="" alt="" width="672" /></p>
<p>The euthanasia rate seems highly correlated with the perceived aggressiveness and size of the breed: Pit Bulls and Rottweilers have the highest rate by far. On the other hand toy breeds that are deemed cute and harmless have very low rates.</p>
<p>Without a few notable exceptions however breed isn’t a real deal breaker or maker when it comes to outcome however.</p>
<div id="hour-and-weekday" class="section level3">
<h3>Hour and Weekday:</h3>
<p>Do more animals get adopted on the weekend? During rush hour? Stay tuned to find out. First we have to extract the information from the <code>DateTime</code> columns. This is easily done using the <code>lubridate</code> package.</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r">train %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">mutate</span>(<span class="dt">hour =</span> <span class="kw">hour</span>(train$DateTime),
           <span class="dt">wday =</span> <span class="kw">wday</span>(train$DateTime)) -&gt;<span class="st"> </span>train</code></pre></div>
<p>Let’s look at the hour and plot the count by hour with fill colors based on outcome.</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class="kw">ggplot</span>(<span class="kw">filter</span>(train), <span class="kw">aes</span>(<span class="dt">x =</span> hour, <span class="dt">fill =</span> OutcomeType)) +
<span class="st">    </span><span class="kw">geom_bar</span>(<span class="dt">stat =</span> <span class="st">&quot;count&quot;</span>) +<span class="st"> </span>
<span class="st">    </span><span class="kw">scale_fill_brewer</span>(<span class="dt">palette =</span> <span class="st">&quot;Set1&quot;</span>) +
<span class="st">    </span><span class="kw">ggtitle</span>(<span class="st">&quot;Outcomes by Hour&quot;</span>)</code></pre></div>
<p><img src="__results___files/figure-html/unnamed-chunk-13-1.png" title="" alt="" width="672" /></p>
<p>Interesting. The bulk of activity happens, as expected, during the day - roughly between 8 and 19 (or 7pm). A lot of transfers happen at 0 - this might be some sort of artefact, but it is worth keeping.</p>
<p>It’s a little hard to see relative frequencies since the bars have different sizes so let’s zoom in to the more popular hours and normalize the lenghs of our bars:</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class="kw">ggplot</span>(<span class="kw">filter</span>(train, hour &gt;<span class="st"> </span><span class="dv">7</span>, hour &lt;<span class="dv">21</span>), <span class="kw">aes</span>(<span class="dt">x =</span> hour, <span class="dt">fill =</span> OutcomeType)) +
<span class="st">    </span><span class="kw">geom_bar</span>(<span class="dt">stat =</span> <span class="st">&quot;count&quot;</span>, <span class="dt">position =</span> <span class="st">&quot;fill&quot;</span>) +<span class="st"> </span>
<span class="st">    </span><span class="kw">scale_fill_brewer</span>(<span class="dt">palette =</span> <span class="st">&quot;Set1&quot;</span>) +
<span class="st">    </span><span class="kw">ggtitle</span>(<span class="st">&quot;Outcomes for Daytime Hours&quot;</span>)</code></pre></div>
<p><img src="__results___files/figure-html/unnamed-chunk-14-1.png" title="" alt="" width="672" /></p>
<p>Hmmm…the adoptions start off strong at 8,then drop precipitously at 9, pick back up at 10 then are roughly stable untill the evening when the adoption rates rise significantly. Also note how most euthanasias are done in the morning - at 8 am and 10 am, respectively. I wonder why.</p>
<p>Let’s also look at outcomes by day of week:</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class="kw">ggplot</span>(train, <span class="kw">aes</span>(<span class="dt">x =</span> wday, <span class="dt">fill =</span> OutcomeType)) +
<span class="st">    </span><span class="kw">geom_bar</span>(<span class="dt">stat =</span> <span class="st">&quot;count&quot;</span>, <span class="dt">position =</span> <span class="st">&quot;fill&quot;</span>) +<span class="st"> </span>
<span class="st">    </span><span class="kw">scale_fill_brewer</span>(<span class="dt">palette =</span> <span class="st">&quot;Set1&quot;</span>) +
<span class="st">    </span><span class="kw">ggtitle</span>(<span class="st">&quot;Outcomes by Day of Week&quot;</span>)</code></pre></div>
<p><img src="__results___files/figure-html/unnamed-chunk-15-1.png" title="" alt="" width="672" /></p>
<p>It looks like more animals are adopted and less animals are transfered on the weekend.</p>
<p>Based on the graphs above it looks like hour and weekday are important features for predicting outcome but, to be honest, I would have liked it better if these features were not included. This is because <code>DateTime</code> has no predictive power in a real life situation. All it tell us is how the shelters operate - more euthanasias in the morning, more adoptions later in the day etc. <strong>If you wanted to predict the outcome of a specific animal, say, a 9 month old beagle named Jimmy, you wouldn’t know a priori what time the outcome would happen.</strong> So these features, while having predictive power in the Kaggle contest, are rather useless in real life situations.</p>
</div>
</div>
<div id="colors" class="section level2">
<h2>Colors:</h2>
<p>Let’s look at colors too. Perhaps surprisingly it seems like color doesn’t play too big a role once we control for animal type. Perhaps the most interesting aspect of the plot below is that cats are returned to owners at a significantly lower rate than dogs.</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class="co"># color doesn&#39;t seem to matter:</span>
<span class="kw">strsplit</span>(<span class="dt">x =</span> train$Color, <span class="dt">split =</span> <span class="st">&quot;/&quot;</span>) %&gt;%<span class="st"> </span><span class="kw">sapply</span>(function(x){x[<span class="dv">1</span>]}) -&gt;<span class="st"> </span>train$color1

<span class="co">#pplot(color1)</span>

<span class="co">#CATS:</span>
cats &lt;-<span class="st"> </span><span class="kw">filter</span>(train, AnimalType ==<span class="st"> &quot;Cat&quot;</span>)

cats %&gt;%<span class="st"> </span><span class="kw">count</span>(color1) %&gt;%<span class="st"> </span><span class="kw">arrange</span>(<span class="kw">desc</span>(n)) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">filter</span>(n &gt;<span class="st"> </span><span class="dv">400</span>) -&gt;<span class="st"> </span>cat_colors

cats$color1[!(cats$color1 %in%<span class="st"> </span>cat_colors$color1)] &lt;-<span class="st"> &quot;Other&quot;</span>

<span class="kw">ggplot</span>(cats, <span class="kw">aes</span>(<span class="dt">x =</span> color1, <span class="dt">fill =</span> OutcomeType)) +
<span class="st">    </span><span class="kw">geom_bar</span>(<span class="dt">stat =</span> <span class="st">&quot;count&quot;</span>, <span class="dt">position =</span> <span class="st">&quot;fill&quot;</span>, <span class="dt">width =</span> <span class="fl">0.7</span>) +<span class="st"> </span>
<span class="st">    </span><span class="kw">ggtitle</span>(<span class="st">&quot;Cat Colors by Outcome&quot;</span>) +
<span class="st">    </span><span class="kw">coord_flip</span>() +
<span class="st">    </span><span class="kw">scale_fill_brewer</span>(<span class="dt">palette =</span> <span class="st">&quot;Set1&quot;</span>) +
<span class="st">    </span><span class="kw">theme</span>(<span class="dt">axis.title =</span> <span class="kw">element_blank</span>(), 
          <span class="dt">legend.position =</span> <span class="st">&quot;none&quot;</span>) -&gt;<span class="st"> </span>g3


<span class="co">#DOGS:</span>
cats &lt;-<span class="st"> </span><span class="kw">filter</span>(train, AnimalType ==<span class="st"> &quot;Dog&quot;</span>)

cats %&gt;%<span class="st"> </span><span class="kw">count</span>(color1) %&gt;%<span class="st"> </span><span class="kw">arrange</span>(<span class="kw">desc</span>(n)) %&gt;%<span class="st"> </span>
<span class="st">    </span><span class="kw">filter</span>(n &gt;<span class="st"> </span><span class="dv">400</span>) -&gt;<span class="st"> </span>cat_colors

cats$color1[!(cats$color1 %in%<span class="st"> </span>cat_colors$color1)] &lt;-<span class="st"> &quot;Other&quot;</span>

<span class="kw">ggplot</span>(cats, <span class="kw">aes</span>(<span class="dt">x =</span> color1, <span class="dt">fill =</span> OutcomeType)) +
<span class="st">    </span><span class="kw">geom_bar</span>(<span class="dt">stat =</span> <span class="st">&quot;count&quot;</span>, <span class="dt">position =</span> <span class="st">&quot;fill&quot;</span>, <span class="dt">width =</span> <span class="fl">0.7</span>) +<span class="st"> </span>
<span class="st">    </span><span class="kw">coord_flip</span>() +
<span class="st">    </span><span class="kw">ggtitle</span>(<span class="st">&quot;Dog Colors by Outcome&quot;</span>) +
<span class="st">    </span><span class="kw">scale_fill_brewer</span>(<span class="dt">palette =</span> <span class="st">&quot;Set1&quot;</span>) +
<span class="st">    </span><span class="kw">theme</span>(<span class="dt">axis.title =</span> <span class="kw">element_blank</span>()) -&gt;<span class="st"> </span>g4

<span class="kw">grid.arrange</span>(g3, g4, <span class="dt">nrow =</span> <span class="dv">1</span>, <span class="dt">widths =</span> <span class="kw">c</span>(<span class="dv">3</span>,<span class="dv">5</span>))</code></pre></div>
<p><img src="__results___files/figure-html/unnamed-chunk-16-1.png" title="" alt="" width="672" /></p>
<div id="thanks-for-reading" class="section level3">
<h3>Thanks for reading!</h3>
</div>
</div>



</div>

<script>

// add bootstrap table styles to pandoc tables
$(document).ready(function () {
  $('tr.header').parent('thead').parent('table').addClass('table table-condensed');
});

</script>

<!-- dynamically load mathjax for compatibility with self-contained -->
<script>
  (function () {
    var script = document.createElement("script");
    script.type = "text/javascript";
    script.src  = "https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML";
    document.getElementsByTagName("head")[0].appendChild(script);
  })();
</script>

</body>
</html>
