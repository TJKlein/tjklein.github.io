---
enable_d3: true

---

### Bio  

Currently, I am Director of AI Research at SAP SE in [Berlin, Germany](https://www.google.com/maps/dir/52.5467648,13.4660096/SAP+Münzstr+15/@52.5365216,13.4064717,13z/data=!3m1!4b1!4m9!4m8!1m1!4e1!1m5!1m1!1s0x47a851e1dab74057:0xae8d4cd859f4c58a!2m2!1d13.406999!2d52.5242668).
Prior to joining SAP, I was a postdoctoral research fellow at Harvard Medical School, Brigham & Women's Hospital, Boston, in the group of [Sandy Wells](https://lmi.med.harvard.edu/people/william-wells). At the same time, I was a postdoctoral research associate in the Computer Science and Artificial Intelligence Laboratory (CSAIL) at MIT, working with the group of [Polina Goland](https://people.csail.mit.edu/polina/index.html). During that time I was conducting research on large-scale machine learning and optimization technologies for discriminative pattern discovery of genetically driven imaging biomarkers.
I obtained my Ph.D. from Technical University of Munich (TUM) at the intersection of medical imaging and machine learning (raw ultraound data processing for applications such as early detection of Parkinson's disease), advised by [Nassir Navab](http://campar.in.tum.de/Main/NassirNavab).

### Research

My research interests lie at the intersection of machine learning, computer vision, and natural language processing. Although not my current main focus, I am very much interested in machine learning in the medical domain and hope to pursue this further at some point.

*Internship Position:* Our research team is continuously hiring students and interns. If you're a Ph.D. student interested in a research internship working in Berlin, please send an email with your CV and research interests. Right now, we are particularly looking for interns for medical imaging (self-supervised learning), NLP (commonsense reasoning), and privacy-preserving ML. Check out our research team's [blog](https://medium.com/sap-machine-learning-research) to learn more about current research activity there.

<div id="chart"></div>
<script src="https://d3js.org/d3.v5.min.js"></script>
<script>
  // (c) by Tassilo Klein, 2022 (https://tjklein.github.io)

var index = -1
d3.json("https://raw.githubusercontent.com/TJKlein/tjklein.github.io/master/data/highres_transformer.json",   function(data) {

const t = d3.transition()
      .duration(750);

var margin = {top: 100, right: 50, bottom: 50, left: 50}
  , width = 600 - margin.left - margin.right // Use the window's width
  , height = 400 - margin.top - margin.bottom; // Use the window's height
  var svg = d3.select("body").append("svg").attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
  .append("g")
    .attr("transform", "translate(" + margin.left + "," + margin.top + ")");
  
  function preprocess(data) {
    index = index +1
    if (index == data.length) {
      index = 0
    }
    return ({min_domain_x: data[index].data_range.min, max_domain_x: data[index].data_range.max, data: data[index].data_set1})
  }

 function update(myData) {

var max_domain_x = myData.max_domain_x*1.1 // data[0].data_range.max
var min_domain_x = myData.min_domain_x*.4 // data[0].data_range.min


// 1. Add the SVG to the page and employ #2

// X axis: scale and draw:
  var x = d3.scaleLinear()
      .domain([min_domain_x,max_domain_x])     // can use this instead of 1000 to have the max of data: d3.max(data, function(d) { return +d.price })
      .range([0, width]);

var categories = ["1","2","3","4","5","6","7","8","9","10","11","12"];
 
var col_array = d3.quantize(d3.interpolateHcl("#69b36b", "#69b3a2"), categories.length)
  
// Create the Y axis for names
var yName = d3.scaleBand()
    .domain(categories)
    .range([0, height])
    .paddingInner(1)
  svg.append("g")
   // .call(d3.axisLeft(yName));


function kde(kernel, thresholds, data) {
  return thresholds.map(t => [t, d3.mean(data, d => kernel(t - d))]);
}
function epanechnikov(bandwidth) {
  return x => Math.abs(x /= bandwidth) <= 1 ? 0.75 * (1 - x * x) / bandwidth : 0;
}
// number of samples for KDE
var n_kde = 250
// number of samples for binning
var n_hist = 250
// var data_set1 = data[0].data_set1;
// determine number of data samples
var n_data = myData.data.length

var allDensity = []
var allBins = []
// this variable is needed to scale data
var maxData = []
// compute the x scale of the data (x Axis), y axis computation is deferred until after KDE
var x = d3.scaleLinear()
    .domain([min_domain_x,max_domain_x]).nice()
    .range([0, width])
//console.log(d3.count(Array.from(["first","second", "third", "fourth", "fiveth", "sixth"])));
for (var i = 0; i < categories.length; i++) {
  var key = categories[i]
  var mean = d3.randomUniform(30,90)()
  var variance = d3.randomUniform(5,10)()
  //var data_set = d3.range(n_data).map(function(d) { return d3.randomNormal(mean, variance)() })
  var data_set = myData.data[i]
   //console.log(data_set)
  var kde_thresholds = x.ticks(n_kde)
  var hist_thresholds = x.ticks(n_hist)
  var bandwidth = 0.35
  var density = kde(epanechnikov(bandwidth), kde_thresholds, data_set)


  var bins = d3.histogram()
    .domain(x.domain())
    .thresholds(hist_thresholds)
  (data_set)


  allDensity.push({key: key, density: density})
  allBins.push({key: key, bins: bins})


  // push data for finding the max value
  density.forEach((element) => { maxData.push(element[1])});
}

// determine max value to scale nicely
maxData = d3.max(maxData)
var y = d3.scaleLinear()
    .domain([0, maxData*categories.length*(1-0.2)])
    .range([ height, 0]);
  

// Add areas
  var u = svg.selectAll("areas")
    .data(allDensity)
    .enter()
    .append("path")
   .attr("stroke", function(d){ 
        return(col_array[d.key-1])})
      .attr("transform", function(d){
        return("translate(0," + (yName(d.key)-height) +")" )})
      .datum(function(d){return(d.density)})
      //.attr("fill", "69b3a2")//"#69b3a2")
      .attr("fill-opacity","0.0")
      .attr("stroke-opacity", "0.0")
      //.transition()
       // .style("stroke-opacity","1.0")
       // .duration(5000)
      //.attr("fill-opacity","0.5")
      .attr("stroke-width", 1)
      .attr("d",  d3.line()
          .curve(d3.curveBasis)
          .x(function(d) { return x(d[0]); })
          .y(function(d) { return y(d[1]); })
      )
      .transition()
        .ease(d3.easeLinear)
      .transition()
        .style("stroke-opacity","1.0")
        .duration(1000)
      .transition()
        .style("fill", "69b3a2")
        .style("fill-opacity","0.5")
        .duration(2000)
      .transition()
            .style("opacity", 0.0)
            .duration(10000);



}
  
  // inspired by:
  // https://bl.ocks.org/d3noob/464c92429ac05c6a19a1
 
  d3.interval(() => {
  update(preprocess(data));
}, 2000);

  });

 </script>

### News

[2021.08] Two papers accepted at [EMNLP 2021](https://2021.emnlp.org/) on Contrastive Self-Supervised Learning for Commonsense Reasoning - [arXiv](https://arxiv.org/abs/2109.05105), [code](https://github.com/SAP-samples/emnlp2021-contrastive-refinement/) and [arXiv](https://arxiv.org/abs/2109.05108), [code] (https://github.com/SAP-samples/emnlp2021-attention-contrastive-learning/)

[2021.04] Acceptance of co-organized at [ICML 2021 workshop](https://icml21ssl.github.io/index.html) on Self-Supervised Learning for Reasoning and Perception 

[2021.02] Paper accepted at [IPMI 2021](https://ipmi2021.org/) on self-supervised representation learning for medical imaging (acceptance rate 30.0%)

[2020.09] Presentation on commonsense reasoning in AI: [video](https://youtu.be/AdA6aJpxFfM?t=2457), [blog](https://medium.com/sap-machine-learning-research/common-sense-still-not-common-in-ai-9d68f431e17f?source=friends_link&sk=667a5243eba0e5c19b28941ce8bd1082).

[2020.04] Paper accepted at [ACL 2020](https://acl2020.org/) on contrastive self-supervised commonsense reasoning (acceptance rate of 17.6%) - [arXiv](https://arxiv.org/abs/2005.00669), [code](https://github.com/SAP-samples/acl2019-commonsense-reasoning), [video](http://slideslive.com/38929108).

[2020.02] Paper accepted at  [NeuroImage](https://www.journals.elsevier.com/neuroimage) - [link](https://www.sciencedirect.com/science/article/pii/S2213158220300231)

[2019.10.20] Paper on Multi-Domain Learning accepted at [ICCV 2019](http://iccv2019.thecvf.com/) (acceptance rate 25.0%) - [arXiv](https://arxiv.org/abs/1905.06242)

[2019.05.14] Short-paper on commonsense reasoning accepted at [ACL 2019](http://www.acl2019.org/EN/index.xhtml) (acceptance rate 18.2%) - [arXiv](https://arxiv.org/abs/1905.13497), [code](https://github.com/SAP-samples/acl2019-commonsense-reasoning)

[2019.02.25] Paper accepted at [CVPR 2019](http://cvpr2019.thecvf.com/) (acceptance rate 25.2%) -[link](http://openaccess.thecvf.com/content_CVPR_2019/html/Ostapenko_Learning_to_Remember_A_Synaptic_Plasticity_Driven_Framework_for_Continual_CVPR_2019_paper.html), [code](https://github.com/SAP/machine-learning-dgm)

[2017.02.01] Paper accept at [NeuroImage](https://www.journals.elsevier.com/neuroimage) - [arXiv](https://arxiv.org/abs/1702.08192), [code](https://github.com/TJKlein/DeepNAT)


*[last update: 09/15/2021]*
