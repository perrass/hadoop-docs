# 明确下各学习线

### All algorithms/tools you known

1. 回归: Regression
2. 分类: Logistic Regression/CART/Adaboost/Random Forest/GBDT/SVM/Navie Bayesian/Neural Network
3. 实例: KNN
4. 聚类: 
   1. Distanced-based: k-means/k-medians/k-medoids/kernel k-means
   2. Hierarchical: agglomerative/divisive/birch/cure/chameleon
   3. Density-based: DBScan/optics
   4. Grid-based: sting/clique
   5. Others (from skicit-learn):
5. 自然语言处理: 语言模型, MaxEnt/CRF, pLSA/LDA, w2v, CNN/RNN
6. 推荐: CF-users/CF-items
7. 概率图模型: MC/HMM/Bayesian Net/Markov Net/MCMC
8. 图:
9. 深度学习: CNN/RNN/LSTM/GRU/Seq2Seq/GANs/Autoencoder
10. 工具箱:
  1. Distribution: Bernoulli/Gaussian/Dirichlet
  2. Loss: Huber Loss/Log Loss/RSS
  3. Regularization: Lasso/Ridge/Elastic Net
  4. Distance/Similarity:
  5. Sampling: Gibbs Sampling
  6. Algorithms: Gradient Descent/EM/Back Propagation
  7. Data Preprocessing: One-hot Encoding/Deskew/Imputation/Standardization/Normalization/Scalization
  8. Text Preprocessing: TF-IDF/Word2Vec/GloVec
  9. Image Preprocessing: Upsampling/Downsampling
  10. Deep Learning: Dropout/Batch Normalization/Layer Normalization/ReLU/Padding/Deconv

### 早（机器学习线）

**除了1，2比较明晰，后面的并不绝对，MLAPP是永远的主线**

#### 1. 以ML和MLND为线索，看MLAPP/统计学习基础/机器学习 —— 8.1

1. Regression (5.12) $\color{red}{\mathbf {Bigoo!}}$
2. Logistic Regression (5.19) $\color{red}{\mathbf {Bigoo!}}$
3. Ensemble Learning: Tree Adaboost GBDT Random Forest(5.26 - 6.4, 机器学习技法)
4. SVM (6.11) (机器学习技法)
5. Bayesian (6.25)(including Bayesian Statistics on Youtube)
6. Clustering(7.2 clustering analysis in data mining 可以上班时间看不用放到早上，早上)
7. PCA (7.9)

#### 2. CMU ML —— 1.1

学习一些高级技能点或者是补全整个机器学习的体系，和能够推公式的数学基础，统计 -> 10-725 -> 10-715 -> 10-702  -> 10-605 -> PGM -> MCMC/HMM

1. 36-705: Intermediate Statistics + All of statistics
2. 10-715: Advanced Introduction to Machine Learning $\color{red}{\mathbf {This~is~the~guide~line~of~all}}$
   1. Neural Network
   2. Clustering, K-means, EM, GMM, PCA
   3. Convex Optimization, SVM, Kernels, Gaussian Processes
   4. ICA
   5. HMM, PGM, MCMC
   6. Nonlinear dim reduction
3. 10-702: Statistical Machine Learning + The elements of statistical learning $\color{red}{\mathbf {The~third~course}}$
   1. Advanced and Statistical topics of 10-715
4. 15-780: Graduate Artificial Intelligence
   1. Search
   2. Linear / Integer Programming
   3. Machine / Deep Learning
   4. Probabilistic Modeling
   5. Game theory
   6. Social Choice
5. 10-725: Convex Optimization
   1. Fundamentals
   2. First-order methods
   3. Optimality and Duality $\color{red}{\mathbf {might~be~enough}}$
   4. Second-order methods
   5. Special topics
6. 10-605: Machine Learning with Large Datasets (Spark)
   1. Hadoop
   2. Parallel Perceptron
   3. SGD
   4. Graphs
   5. LDA
7. Coursera: Probabilistic Graphical Model
   1. Bayesian Net / Markov Net
   2. MAP / MCMC / Sampling
   3. MLE / BIC / EM 
8. 10-708: Probabilistic Graphical Model (Advanced courses compared with Coursera)
9. Harvard AM207 Stochastic Methods for Data Analysis, Inference and Optimization (Basic courses compared with Coursera)
   1. More details about MC, Bayesian and MCMC
   2. Time series / HMM / Gaussian Process

#### 3. 源码学习 (写包玩！！！) ——3.1

以scikit-learn User Guide为主线

1. scikit-learn
2. xgboost
3. libsvm
4. lightboost
5. spark ml/mllib

#### 4. 进一步数学学习+课后题 —— ?

* MLAPP
* Convex Optimization
* All of Statistics

### 白天（深度学习线）

#### 1. DLND 课程(5.27) 

$\color{red}{\mathbf {Bigoo!}}$

#### 1. 机器学习

* MLND (无监督6.10)
* MLND (强化7.8)
* Clustering (5.29 - 6.25)
* Reinforment Learning: Blog and Materials (6.25 - 7.8)
* NLP (Text Retrivel, Text Mining)

#### 2. DLND review (5.31 - 7.27)

two course weeks per week

* DLND courses
* Siraj youtube live/code/materials
* DLND bilibili channel

#### 3. 系统学习深度学习(7.28 - 1.1)

* DL/Deep Learning Book Club
* CS224n
* CS231n
* Oxford Deepnlp

#### 4. Papers/Codes/Projects

### 晚上（算法线）

听课（DL, CV, ML），现在基本结束了，所以以Intro to algorithms 为主

对算法来说这五门课足够了

#### 1. 6.006 Introduction  to Algorithms & Hacker-rank —— 9.12

#### 2. 6.046 Design and Analysis of Algorithms & Hacker-rank—— 12.12

#### 3. Leet-Code -> Code-wars —— 3.1

#### 4. Advanced Algorithms —— ? Finished with MLAPP

1. CS229r Algorithms for Big Data
2. 6.854 Advanced Algorithms
3. 6.851 Advanced Data Structure

### 周末（程序/项目线, 6.3开始）

1. Java: Java Coursera & Thinking in Java & CS106A-> edX MIT & CS108 & Design Pattern
2. Scala: Scala Coursera -> Functional Programming in Scala
3. C/C++: CS106B -> CS107 & CS107(2014)-> CMU 15-418 & 并行编程入门
4. CS: Operation System & Udacity-Intro to Algorithms (GraphX) & 编程设计 & CSE 8803





