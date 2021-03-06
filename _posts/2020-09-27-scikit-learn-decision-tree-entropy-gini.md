---
layout: page
title: "의사결정나무 (Decision Tree)와 Entropy, 그리고 Gini 계수"
description: "의사결정나무 (Decision Tree)와 Entropy, 그리고 Gini 계수에 대해 알아보겠습니다."
headline: "의사결정나무 (Decision Tree)와 Entropy, 그리고 Gini 계수에 대해 알아보겠습니다."
categories: scikit-learn
tags: [python, tensorflow, scikit-learn, Decision Tree, 의사 결정 나무, Random Forest, 텐서플로우, data science, 데이터 분석, 딥러닝, 딥러닝 자격증, 머신러닝, 빅데이터, 테디노트]
comments: true
published: true
typora-copy-images-to: ../images/2020-09-27
use_math: true
---



Decision Tree는 Random Forest Ensemble 알고리즘의 기본이 되는 알고리즘이며, **Tree 기반 알고리즘**입니다. 의사결정나무 혹은 결정트리로 불리우는 이 알고리즘은 머신러닝의 학습 결과에 대하여 시각화를 통한 직관적인 이해가 가능하다는 것이 큰 장점입니다. 더불어, **Random Forest Ensemble 알고리즘은 바로 이 Decision Tree 알고리즘의 앙상블 (Ensemble) 알고리즘**인데, Random Forest 앙상블 알고리즘이 사용성은 쉬우면서 성능까지 뛰어나 캐글 (Kaggle.com)과 같은 데이터 분석 대회에서 Baseline 알고리즘으로 많이 활용되고 있습니다.

이번 실습에서는 Decision Tree 알고리즘에 대하여 시각화, **Entropy와 Gini 계수에 대한 상세한 이해**를 돕고자 만들어진 튜토리얼 이며, 실습 후반부에는 자주 사용되는 Hyperparameter 에 대한 소개도 진행합니다.

Random Forest 의 Hyperparameter와 겹치는 부분이 많기 때문에, 본 실습을 통해 Hyperparameter에 대하여 숙지해 두시면 자동으로 Random Forest 알고리즘의 Hyperparameter에 대한 이해까지 할 수 있게되는 1석 2조의 효과를 보실 수 있습니다.



## 코드

![Colab으로 열기](../images/2020-09-27/colab_logo_32px.png) [Colab으로 열기](https://colab.research.google.com/github/teddylee777/machine-learning/blob/master/10-scikit-learn/08-%EA%B2%B0%EC%A0%95%ED%8A%B8%EB%A6%AC%20(Decistion%20Tree).ipynb)

![GitHub](../images/2020-09-27/GitHub-Mark-32px.png) [GitHub에서 소스보기](https://github.com/teddylee777/machine-learning/blob/master/10-scikit-learn/08-%EA%B2%B0%EC%A0%95%ED%8A%B8%EB%A6%AC%20(Decistion%20Tree).ipynb)

<br/>

<body>

<div class="border-box-sizing" id="notebook" >
<div class="container" id="notebook-container">
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="kn">from</span> <span class="nn">IPython.display</span> <span class="k">import</span> <span class="n">Image</span>
</pre></div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h1 id="결정트리-or-의사결정나무-(Decision-Tree)">결정트리 or 의사결정나무 (Decision Tree)</h1>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>결정트리를 가장 단수하게 표현하자면, <strong>Tree 구조를 가진 알고리즘</strong>입니다.</p>
<p>의사결정나무는 데이터를 분석하여 데이터 사이에서 패턴을 예측 가능한 규칙들의 조합으로 나타내며, 이 과정을 시각화 해 본다면 마치 <strong>스무고개</strong> 놀이와 비슷합니다.</p>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">Image</span><span class="p">(</span><span class="n">url</span><span class="o">=</span><span class="s1">'https://miro.medium.com/max/2960/1*dc_342kIsHCzuko1TtyEGQ.png'</span><span class="p">,</span> <span class="n">width</span><span class="o">=</span><span class="mi">500</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">

<div class="output_html rendered_html output_subarea output_execute_result">
<img src="https://miro.medium.com/max/2960/1*dc_342kIsHCzuko1TtyEGQ.png" width="500"/>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>결정트리의 기본 아이디어는 sample이 가장 섞이지 않은 상태로 완전히 분류되는 것, 다시 말해서 <strong>엔트로피(Entropy)를 낮추도록</strong> 만드는 것입니다.</p>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="엔트로피-(Entropy)">엔트로피 (Entropy)</h2>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>엔트로피는 쉽게 말해서 <strong>무질서한 정도를 정량화(수치화)한 값</strong>입니다.</p>
<p>다음은 <strong>엔트로피 지수를 방이 어질러있는 정도를 예시로 들어 표현</strong>되었습니다.</p>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">Image</span><span class="p">(</span><span class="n">url</span><span class="o">=</span><span class="s1">'https://image.slidesharecdn.com/entropyandthe2ndlaw-120327062903-phpapp02/95/103-entropy-and-the-2nd-law-3-728.jpg?cb=1335190079'</span><span class="p">,</span> <span class="n">width</span><span class="o">=</span><span class="mi">500</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">

<div class="output_html rendered_html output_subarea output_execute_result">
<img src="https://image.slidesharecdn.com/entropyandthe2ndlaw-120327062903-phpapp02/95/103-entropy-and-the-2nd-law-3-728.jpg?cb=1335190079" width="500"/>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h3 id="엔트로피-수식의-이해">엔트로피 수식의 이해</h3>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="kn">import</span> <span class="nn">numpy</span> <span class="k">as</span> <span class="nn">np</span>
<span class="kn">import</span> <span class="nn">pandas</span> <span class="k">as</span> <span class="nn">pd</span>
<span class="kn">import</span> <span class="nn">matplotlib.pyplot</span> <span class="k">as</span> <span class="nn">plt</span>
<span class="kn">import</span> <span class="nn">seaborn</span> <span class="k">as</span> <span class="nn">sns</span>
</pre></div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="c1"># 샘플데이터를 생성합니다.</span>
<span class="n">group_1</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">array</span><span class="p">([</span><span class="mf">0.3</span><span class="p">,</span> <span class="mf">0.4</span><span class="p">,</span> <span class="mf">0.3</span><span class="p">])</span>
<span class="n">group_2</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">array</span><span class="p">([</span><span class="mf">0.7</span><span class="p">,</span> <span class="mf">0.2</span><span class="p">,</span> <span class="mf">0.1</span><span class="p">])</span>
<span class="n">group_3</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">array</span><span class="p">([</span><span class="mf">0.01</span><span class="p">,</span> <span class="mf">0.01</span><span class="p">,</span> <span class="mf">0.98</span><span class="p">])</span>
</pre></div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">fig</span><span class="p">,</span> <span class="n">axes</span> <span class="o">=</span> <span class="n">plt</span><span class="o">.</span><span class="n">subplots</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">3</span><span class="p">)</span>
<span class="n">fig</span><span class="o">.</span><span class="n">set_size_inches</span><span class="p">(</span><span class="mi">12</span><span class="p">,</span> <span class="mi">4</span><span class="p">)</span>
<span class="n">axes</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span><span class="o">.</span><span class="n">bar</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">arange</span><span class="p">(</span><span class="mi">3</span><span class="p">),</span> <span class="n">group_1</span><span class="p">,</span> <span class="n">color</span><span class="o">=</span><span class="s1">'blue'</span><span class="p">)</span>
<span class="n">axes</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span><span class="o">.</span><span class="n">set_title</span><span class="p">(</span><span class="s1">'Group 1'</span><span class="p">)</span>
<span class="n">axes</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span><span class="o">.</span><span class="n">bar</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">arange</span><span class="p">(</span><span class="mi">3</span><span class="p">),</span> <span class="n">group_2</span><span class="p">,</span> <span class="n">color</span><span class="o">=</span><span class="s1">'red'</span><span class="p">)</span>
<span class="n">axes</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span><span class="o">.</span><span class="n">set_title</span><span class="p">(</span><span class="s1">'Group 2'</span><span class="p">)</span>
<span class="n">axes</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span><span class="o">.</span><span class="n">bar</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">arange</span><span class="p">(</span><span class="mi">3</span><span class="p">),</span> <span class="n">group_3</span><span class="p">,</span> <span class="n">color</span><span class="o">=</span><span class="s1">'green'</span><span class="p">)</span>
<span class="n">axes</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span><span class="o">.</span><span class="n">set_title</span><span class="p">(</span><span class="s1">'Group 3'</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">show</span><span class="p">()</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">
<div class="prompt"></div>
<div class="output_png output_subarea">
<img src="data:image/png;base64,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
"/>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">Image</span><span class="p">(</span><span class="n">url</span><span class="o">=</span><span class="s1">'https://miro.medium.com/max/1122/0*DkWdyGidNSfdT1Nu.png'</span><span class="p">,</span> <span class="n">width</span><span class="o">=</span><span class="mi">350</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">
<div class="output_html rendered_html output_subarea output_execute_result">
<img src="https://miro.medium.com/max/1122/0*DkWdyGidNSfdT1Nu.png" width="350"/>
</div>

</div>
</div>
</div>
</div>

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="c1"># entropy를 구현합니다.</span>
<span class="k">def</span> <span class="nf">entropy</span><span class="p">(</span><span class="n">x</span><span class="p">):</span>
    <span class="k">return</span> <span class="p">(</span><span class="o">-</span><span class="n">x</span><span class="o">*</span><span class="n">np</span><span class="o">.</span><span class="n">log2</span><span class="p">(</span><span class="n">x</span><span class="p">))</span><span class="o">.</span><span class="n">sum</span><span class="p">()</span>
</pre></div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h3 id="Entropy-계산-및-시각화">Entropy 계산 및 시각화</h3>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">entropy_1</span> <span class="o">=</span> <span class="n">entropy</span><span class="p">(</span><span class="n">group_1</span><span class="p">)</span>
<span class="n">entropy_2</span> <span class="o">=</span> <span class="n">entropy</span><span class="p">(</span><span class="n">group_2</span><span class="p">)</span>
<span class="n">entropy_3</span> <span class="o">=</span> <span class="n">entropy</span><span class="p">(</span><span class="n">group_3</span><span class="p">)</span>

<span class="nb">print</span><span class="p">(</span><span class="n">f</span><span class="s1">'Group 1: </span><span class="si">{entropy_1:.3f}</span><span class="se">\n</span><span class="s1">Group 2: </span><span class="si">{entropy_2:.3f}</span><span class="se">\n</span><span class="s1">Group 3: </span><span class="si">{entropy_3:.3f}</span><span class="s1">'</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">
<div class="prompt"></div>
<div class="output_subarea output_stream output_stdout output_text">
<pre>Group 1: 1.571
Group 2: 1.157
Group 3: 0.161
</pre>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">plt</span><span class="o">.</span><span class="n">figure</span><span class="p">(</span><span class="n">figsize</span><span class="o">=</span><span class="p">(</span><span class="mi">5</span><span class="p">,</span> <span class="mi">5</span><span class="p">))</span>
<span class="n">plt</span><span class="o">.</span><span class="n">bar</span><span class="p">([</span><span class="s1">'Group 1'</span><span class="p">,</span> <span class="s1">'Group 2'</span><span class="p">,</span> <span class="s1">'Group 3'</span><span class="p">],</span> <span class="p">[</span><span class="n">entropy_1</span><span class="p">,</span> <span class="n">entropy_2</span><span class="p">,</span> <span class="n">entropy_3</span><span class="p">])</span>
<span class="n">plt</span><span class="o">.</span><span class="n">title</span><span class="p">(</span><span class="s1">'Entropy'</span><span class="p">,</span> <span class="n">fontsize</span><span class="o">=</span><span class="mi">15</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">show</span><span class="p">()</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">
<div class="prompt"></div>
<div class="output_png output_subarea">
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAT8AAAFBCAYAAAABjqgaAAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAALEgAACxIB0t1+/AAAADl0RVh0U29mdHdhcmUAbWF0cGxvdGxpYiB2ZXJzaW9uIDIuMi4zLCBodHRwOi8vbWF0cGxvdGxpYi5vcmcvIxREBQAAE3pJREFUeJzt3X+wXHd53/H3BwmHEn6kQRdKLAu5QUwqMjaQiwMTGptCEtmkFgQSLJK4pk6USTE0E1ripq3tmj9ioBOYNCZGMY4aGuQqCRDFCAxDIEprCyQXcCwbU41t8K1JJLBj6tBgBE//2FVmWVba1b1ndffq+37N7Nxzvue55zz3jPS537O7Z2+qCklqzWOWuwFJWg6Gn6QmGX6SmmT4SWqS4SepSYafpCYZflqUJFclqWM8fu4E9vPjSX5lmr1Ko6xe7ga0oj0MbBoxfvAE9vHjwKuAd3TSkTQhw09LcaSq9p6MAyVZBayqqkdPxvF06vOyV1ORZH3/EvhnkrwrycNJFpL8pySP6ddcBbwReMbAJfP2/rbtSfYneXmSA8DfAT/c3/acJB9L8rUkDyX5gyRPG3Hs1yR5T5L/m+RQkisHap7drzl3qO8nJHkkyRumfY60vAw/LUmS1cOPoZK3Ao/Qu7T9b8AV/WWA64H3An8FvLD/ePPA967vf/9vABcA9yaZAz4BPB54DfB64Fzgo0lOGzr224Cv9Y/3u8CVSV4HUFUHgL3Aa4e+56eBx/b70inMy14txVOAbwwPJjlzYHVPVb2xv/zRJJuAnwJ2VtVCki8BXz/G5fNTgJdW1WcG9n1Nf/Enquqr/bHPA58EXgnsGPj+A1X1S/3lm5M8Ffj1JL9TVd8C3g28I8llVfVIv+61wJ9W1ZcnPgtakZz5aSkeBp4/4vHAQM1Hhr7nTmDthPv/P4PB13cO8JGjwQdQVZ8C7gNeNFT7/qH19wHfN3D8G/tffxogyff39/F7E/anFcyZn5biSFXtH7UhydHFvxna9CjwuAn3/9cjxp4OHDhG7fcOjR06xvrTgS9W1SNJdtKb7f0ecAm9S/APT9ifVjBnfpploz5v7UvAU0eMPw14cGhsuO7o+pcGxq4H/mmSDcDFwO9X1TcX0atWGMNPy+1EZoLQe27vJ5I88ehAkufTe3HkfwzVvmJo/afoBd/C0YGqugX4HHADsA7YfgK9aAXzsldLsTrJC0aM338C+/gc8LQklwB3AF+uqvuOU/+bwC/TewHjLcATgGuAvwT+eKj22Une1R//UeBS4F/3X+wY9G56rwzfWlWfO4HetYI589NSPBm4dcRj+O0jx7OT3mzrrcA+4KrjFVfVYeDF9N73twO4FvgL4MdGvAH6TcCT6IXfL9F7G81vj9jtB/pfbziBvrXCxY+x16kmyXrgXuCfV9VNE9T/K3rh+32DryLr1OZlr5rVD8lnAb8ObDf42uJlr1p2FXATcBfwH5e3FZ1sXvZKapIzP0lNMvwkNWnZXvBYs2ZNrV+/frkOL+kUddttt325qubG1S1b+K1fv579+0feFipJi5bkC5PUedkrqUmGn6QmjQ2/JDf0PwL8juPUnJfkM0kOJPnzbluUpO5NMvPbzui/0AVAku8B3glcWFXPpv/BkJI0y8aGX1Xt4Ts/J23Qa4D3VdUX+/XDHyApSTOni+f8ngX8wySfSHJbkos72KckTVUXb3VZDfwQ8BLgHwC3JtlbVZ8fLkyyFdgKsG7dug4OLUmL08XMbwH4cFX9bf8vXu0Bzh5VWFXbqmq+qubn5sa+B1GSpqaL8PsTen8DYXWSx9P7w9J3dbBfSZqasZe9SXYA5wFrkiwAV9L7o85U1XVVdVeSDwO3A98Crq+qY74tRpJmwdjwq6otE9S8jd7fQJCkFWFFfZLz+ss/uNwtzLT7rnnZcrcgrRje3iapSYafpCYZfpKaZPhJapLhJ6lJhp+kJhl+kppk+ElqkuEnqUmGn6QmGX6SmmT4SWqS4SepSYafpCYZfpKaZPhJapLhJ6lJhp+kJhl+kppk+ElqkuEnqUmGn6QmGX6SmjQ2/JLckORQkjvG1D0/yTeTvKq79iRpOiaZ+W0HNh2vIMkq4C3AzR30JElTNzb8qmoP8OCYstcDfwwc6qIpSZq2JT/nl+R04BXAdUtvR5JOji5e8HgH8GtV9c1xhUm2JtmfZP/hw4c7OLQkLc7qDvYxD9yYBGANcEGSI1X1geHCqtoGbAOYn5+vDo4tSYuy5PCrqjOPLifZDtw0KvgkaZaMDb8kO4DzgDVJFoArgccCVJXP80lakcaGX1VtmXRnVXXJkrqRpJPEOzwkNcnwk9Qkw09Skww/SU0y/CQ1yfCT1CTDT1KTDD9JTTL8JDXJ8JPUJMNPUpMMP0lNMvwkNcnwk9Qkw09Skww/SU0y/CQ1yfCT1CTDT1KTDD9JTTL8JDXJ8JPUJMNPUpMMP0lNGht+SW5IcijJHcfY/rNJbu8/bklydvdtSlK3Jpn5bQc2HWf7vcC5VXUW8GZgWwd9SdJUrR5XUFV7kqw/zvZbBlb3AmuX3pYkTVfXz/ldCnyo431KUufGzvwmleTF9MLvRcep2QpsBVi3bl1Xh5akE9bJzC/JWcD1wOaq+sqx6qpqW1XNV9X83NxcF4eWpEVZcvglWQe8D/j5qvr80luSpOkbe9mbZAdwHrAmyQJwJfBYgKq6DrgCeArwziQAR6pqfloNS1IXJnm1d8uY7b8A/EJnHUnSSeAdHpKaZPhJalJnb3XRqWP95R9c7hZm3n3XvGy5W9ASOfOT1CTDT1KTDD9JTTL8JDXJ8JPUJMNPUpMMP0lNMvwkNcnwk9Qkw09Skww/SU0y/CQ1yfCT1CTDT1KTDD9JTTL8JDXJ8JPUJMNPUpMMP0lNMvwkNcnwk9SkseGX5IYkh5LccYztSfJbSQ4muT3J87pvU5K6NcnMbzuw6Tjbzwc29B9bgd9ZeluSNF1jw6+q9gAPHqdkM/D71bMX+J4kT++qQUmahi6e8zsduH9gfaE/Jkkzq4vwy4ixGlmYbE2yP8n+w4cPd3BoSVqcLsJvAThjYH0t8MCowqraVlXzVTU/NzfXwaElaXG6CL9dwMX9V31fADxcVV/qYL+SNDWrxxUk2QGcB6xJsgBcCTwWoKquA3YDFwAHga8Br51Ws5LUlbHhV1Vbxmwv4HWddSRJJ4F3eEhqkuEnqUmGn6QmGX6SmmT4SWqS4SepSYafpCYZfpKaZPhJapLhJ6lJhp+kJhl+kppk+ElqkuEnqUmGn6QmGX6SmmT4SWqS4SepSYafpCYZfpKaZPhJapLhJ6lJhp+kJhl+kpo0Ufgl2ZTk7iQHk1w+Yvu6JB9P8ukktye5oPtWJak7Y8MvySrgWuB8YCOwJcnGobL/AOysqucCFwHv7LpRSerSJDO/c4CDVXVPVT0K3AhsHqop4En95ScDD3TXoiR1b/UENacD9w+sLwA/PFRzFfCRJK8Hvht4aSfdSdKUTDLzy4ixGlrfAmyvqrXABcB7knzHvpNsTbI/yf7Dhw+feLeS1JFJwm8BOGNgfS3feVl7KbAToKpuBR4HrBneUVVtq6r5qpqfm5tbXMeS1IFJwm8fsCHJmUlOo/eCxq6hmi8CLwFI8k/ohZ9TO0kza2z4VdUR4DLgZuAueq/qHkhydZIL+2VvBH4xyWeBHcAlVTV8aSxJM2OSFzyoqt3A7qGxKwaW7wR+pNvWJGl6vMNDUpMMP0lNMvwkNcnwk9Qkw09Skww/SU0y/CQ1yfCT1CTDT1KTDD9JTTL8JDXJ8JPUJMNPUpMMP0lNMvwkNcnwk9Qkw09Skww/SU0y/CQ1yfCT1CTDT1KTDD9JTTL8JDXJ8JPUpInCL8mmJHcnOZjk8mPU/EySO5McSPLebtuUpG6tHleQZBVwLfBjwAKwL8muqrpzoGYD8O+AH6mqh5I8dVoNS1IXJpn5nQMcrKp7qupR4EZg81DNLwLXVtVDAFV1qNs2Jalbk4Tf6cD9A+sL/bFBzwKeleR/JtmbZFNXDUrSNIy97AUyYqxG7GcDcB6wFviLJD9YVX/zbTtKtgJbAdatW3fCzUpSVyaZ+S0AZwysrwUeGFHzJ1X1jaq6F7ibXhh+m6raVlXzVTU/Nze32J4lackmCb99wIYkZyY5DbgI2DVU8wHgxQBJ1tC7DL6ny0YlqUtjw6+qjgCXATcDdwE7q+pAkquTXNgvuxn4SpI7gY8D/7aqvjKtpiVpqSZ5zo+q2g3sHhq7YmC5gF/tPyRp5nmHh6QmGX6SmmT4SWqS4SepSYafpCYZfpKaZPhJapLhJ6lJhp+kJhl+kppk+ElqkuEnqUmGn6QmGX6SmmT4SWqS4SepSYafpCYZfpKaZPhJapLhJ6lJhp+kJhl+kppk+ElqkuEnqUkThV+STUnuTnIwyeXHqXtVkkoy312LktS9seGXZBVwLXA+sBHYkmTjiLonAm8APtl1k5LUtUlmfucAB6vqnqp6FLgR2Dyi7s3AW4G/67A/SZqKScLvdOD+gfWF/tjfS/Jc4IyquqnD3iRpaiYJv4wYq7/fmDwGeDvwxrE7SrYm2Z9k/+HDhyfvUpI6Nkn4LQBnDKyvBR4YWH8i8IPAJ5LcB7wA2DXqRY+q2lZV81U1Pzc3t/iuJWmJJgm/fcCGJGcmOQ24CNh1dGNVPVxVa6pqfVWtB/YCF1bV/ql0LEkdGBt+VXUEuAy4GbgL2FlVB5JcneTCaTcoSdOwepKiqtoN7B4au+IYtectvS1Jmi7v8JDUJMNPUpMMP0lNMvwkNcnwk9Qkw09Skww/SU0y/CQ1yfCT1CTDT1KTDD9JTTL8JDXJ8JPUJMNPUpMMP0lNMvwkNcnwk9Qkw09Skww/SU0y/CQ1yfCT1CTDT1KTDD9JTTL8JDVpovBLsinJ3UkOJrl8xPZfTXJnktuTfCzJM7pvVZK6Mzb8kqwCrgXOBzYCW5JsHCr7NDBfVWcBfwS8tetGJalLk8z8zgEOVtU9VfUocCOwebCgqj5eVV/rr+4F1nbbpiR1a5LwOx24f2B9oT92LJcCH1pKU5I0basnqMmIsRpZmPwcMA+ce4ztW4GtAOvWrZuwRUnq3iQzvwXgjIH1tcADw0VJXgr8e+DCqvr6qB1V1baqmq+q+bm5ucX0K0mdmCT89gEbkpyZ5DTgImDXYEGS5wLvohd8h7pvU5K6NTb8quoIcBlwM3AXsLOqDiS5OsmF/bK3AU8A/jDJZ5LsOsbuJGkmTPKcH1W1G9g9NHbFwPJLO+5LkqbKOzwkNWmimZ+k6Vh/+QeXu4WZd981L5vKfp35SWqS4SepSYafpCYZfpKaZPhJapLhJ6lJhp+kJhl+kppk+ElqkuEnqUmGn6QmGX6SmmT4SWqS4SepSYafpCYZfpKaZPhJapLhJ6lJhp+kJhl+kppk+ElqkuEnqUkThV+STUnuTnIwyeUjtn9Xkv/e3/7JJOu7blSSujQ2/JKsAq4Fzgc2AluSbBwquxR4qKqeCbwdeEvXjUpSlyaZ+Z0DHKyqe6rqUeBGYPNQzWbgv/aX/wh4SZJ016YkdWuS8DsduH9gfaE/NrKmqo4ADwNP6aJBSZqG1RPUjJrB1SJqSLIV2NpffSTJ3RMcf5atAb683E0clVP7yQbP9ckxU+cZFnWunzFJ0SThtwCcMbC+FnjgGDULSVYDTwYeHN5RVW0Dtk3S2EqQZH9VzS93Hy3wXJ8cLZ3nSS579wEbkpyZ5DTgImDXUM0u4F/0l18F/FlVfcfMT5JmxdiZX1UdSXIZcDOwCrihqg4kuRrYX1W7gHcD70lykN6M76JpNi1JSxUnaIuXZGv/Ul5T5rk+OVo6z4afpCZ5e5ukJjUXfkmeluS9Se5JcluSW5O84iT38AP94349yb85mcc+mWbkXP9sktv7j1uSnH0yj38yzMh53tw/x59Jsj/Ji07m8RejqfDr33XyAWBPVf3jqvohei/OrB1RO8nbgBbrQeANwH+e4jGW1Qyd63uBc6vqLODNnEJvtYKZOs8fA86uqucA/xK4forH6kRT4Qf8M+DRqrru6EBVfaGq/gtAkkuS/GGSPwU+kp63JbkjyV8meXW/7rwkNx3dR5LfTnJJf/m+JG9J8qn+45nDTVTVoaraB3xjuj/uspqVc31LVT3UX93LiFBY4WblPD8y8Pa272bETQ6zZpq/CWbRs4H/NabmhcBZVfVgklcCzwHOpvfO931J9kxwnK9W1TlJLgbeAfzkUppeoWbxXF8KfGiCfa4kM3Oe+5favwE8FXjZCfwMy6K1md+3SXJtks8m2Tcw/NGqOnp3youAHVX1zar6a+DPgedPsOsdA19f2F3HK9dyn+skL6YXfr924t2vHMt5nqvq/VX1A8DL6T3FMNNaC78DwPOOrlTV64CXAHMDNX87sHysT6Y5wrefu8cNba9jLLdkZs51krPoPQe1uaq+cvy2V5yZOc8DPewBvj/JmuPVLbfWwu/PgMcl+eWBsccfp34P8Ookq5LMAT8KfAr4ArAxvQ9xfTK9f2yDXj3w9dZuWl9xZuJcJ1kHvA/4+ar6/OJ+lJk2K+f5mf0XX0jyPOA0YKZ/0TT1nF9VVZKXA29P8ibgML3fise6FHo/vSn+Z+n9tntTVf0VQJKdwO3A/wY+PfR935Xkk/R+uWwZ3mmSfwTsB54EfCvJrwAbq+qrS/wRZ8asnGvgCnofr/bO/v/NI6fSjfszdJ5fCVyc5BvA/wNePev393uHR8eS3AfMV9VMfSzQqchzfXKcque5tcteSQKc+UlqlDM/SU0y/CQ1yfCT1CTDT1KTDD9JTTL8JDXp/wOUOpSRvHfE7gAAAABJRU5ErkJggg==
"/>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="지니-계수-(Gini-Index)">지니 계수 (Gini Index)</h2>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<ul>
<li>클래쓰들이 공평하게 섞여 있을 수록 <strong>지니 계수</strong>는 올라갑니다.</li>
<li>Decision Tree는 지니 불순도를 낮추는 방향으로 가지치기를 진행합니다.</li>
</ul>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">Image</span><span class="p">(</span><span class="n">url</span><span class="o">=</span><span class="s1">'http://www.learnbymarketing.com/wp-content/uploads/2016/02/gini-index-formula.png'</span><span class="p">,</span> <span class="n">width</span><span class="o">=</span><span class="mi">350</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">

<div class="output_html rendered_html output_subarea output_execute_result">
<img src="http://www.learnbymarketing.com/wp-content/uploads/2016/02/gini-index-formula.png" width="350" />
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="c1"># Gini Index 구현합니다.</span>
<span class="k">def</span> <span class="nf">gini</span><span class="p">(</span><span class="n">x</span><span class="p">):</span>
    <span class="k">return</span> <span class="mi">1</span> <span class="o">-</span> <span class="p">((</span><span class="n">x</span> <span class="o">/</span> <span class="n">x</span><span class="o">.</span><span class="n">sum</span><span class="p">())</span><span class="o">**</span><span class="mi">2</span><span class="p">)</span><span class="o">.</span><span class="n">sum</span><span class="p">()</span>
</pre></div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="c1"># 샘플데이터를 생성합니다.</span>
<span class="n">group_1</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">array</span><span class="p">([</span><span class="mi">50</span><span class="p">,</span> <span class="mi">50</span><span class="p">])</span>
<span class="n">group_2</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">array</span><span class="p">([</span><span class="mi">30</span><span class="p">,</span> <span class="mi">70</span><span class="p">])</span>
<span class="n">group_3</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">array</span><span class="p">([</span><span class="mi">0</span><span class="p">,</span> <span class="mi">100</span><span class="p">])</span>
</pre></div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">fig</span><span class="p">,</span> <span class="n">axes</span> <span class="o">=</span> <span class="n">plt</span><span class="o">.</span><span class="n">subplots</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">3</span><span class="p">)</span>
<span class="n">fig</span><span class="o">.</span><span class="n">set_size_inches</span><span class="p">(</span><span class="mi">12</span><span class="p">,</span> <span class="mi">4</span><span class="p">)</span>
<span class="n">axes</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span><span class="o">.</span><span class="n">bar</span><span class="p">([</span><span class="s1">'Positive'</span><span class="p">,</span> <span class="s1">'Negative'</span><span class="p">],</span> <span class="n">group_1</span><span class="p">,</span> <span class="n">color</span><span class="o">=</span><span class="s1">'blue'</span><span class="p">)</span>
<span class="n">axes</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span><span class="o">.</span><span class="n">set_title</span><span class="p">(</span><span class="s1">'Group 1'</span><span class="p">)</span>
<span class="n">axes</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span><span class="o">.</span><span class="n">bar</span><span class="p">([</span><span class="s1">'Positive'</span><span class="p">,</span> <span class="s1">'Negative'</span><span class="p">],</span> <span class="n">group_2</span><span class="p">,</span> <span class="n">color</span><span class="o">=</span><span class="s1">'red'</span><span class="p">)</span>
<span class="n">axes</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span><span class="o">.</span><span class="n">set_title</span><span class="p">(</span><span class="s1">'Group 2'</span><span class="p">)</span>
<span class="n">axes</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span><span class="o">.</span><span class="n">bar</span><span class="p">([</span><span class="s1">'Positive'</span><span class="p">,</span> <span class="s1">'Negative'</span><span class="p">],</span> <span class="n">group_3</span><span class="p">,</span> <span class="n">color</span><span class="o">=</span><span class="s1">'green'</span><span class="p">)</span>
<span class="n">axes</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span><span class="o">.</span><span class="n">set_title</span><span class="p">(</span><span class="s1">'Group 3'</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">show</span><span class="p">()</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">
<div class="prompt"></div>
<div class="output_png output_subarea">
<img src="data:image/png;base64,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
"/>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">gini_1</span> <span class="o">=</span> <span class="n">gini</span><span class="p">(</span><span class="n">group_1</span><span class="p">)</span>
<span class="n">gini_2</span> <span class="o">=</span> <span class="n">gini</span><span class="p">(</span><span class="n">group_2</span><span class="p">)</span>
<span class="n">gini_3</span> <span class="o">=</span> <span class="n">gini</span><span class="p">(</span><span class="n">group_3</span><span class="p">)</span>

<span class="nb">print</span><span class="p">(</span><span class="n">f</span><span class="s1">'Group 1: </span><span class="si">{gini_1:.3f}</span><span class="se">\n</span><span class="s1">Group 2: </span><span class="si">{gini_2:.3f}</span><span class="se">\n</span><span class="s1">Group 3: </span><span class="si">{gini_3:.3f}</span><span class="s1">'</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">
<div class="prompt"></div>
<div class="output_subarea output_stream output_stdout output_text">
<pre>Group 1: 0.500
Group 2: 0.420
Group 3: 0.000
</pre>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">plt</span><span class="o">.</span><span class="n">figure</span><span class="p">(</span><span class="n">figsize</span><span class="o">=</span><span class="p">(</span><span class="mi">5</span><span class="p">,</span> <span class="mi">5</span><span class="p">))</span>
<span class="n">plt</span><span class="o">.</span><span class="n">bar</span><span class="p">([</span><span class="s1">'Group 1'</span><span class="p">,</span> <span class="s1">'Group 2'</span><span class="p">,</span> <span class="s1">'Group 3'</span><span class="p">],</span> <span class="p">[</span><span class="n">gini_1</span><span class="p">,</span> <span class="n">gini_2</span><span class="p">,</span> <span class="n">gini_3</span><span class="p">])</span>
<span class="n">plt</span><span class="o">.</span><span class="n">title</span><span class="p">(</span><span class="s1">'Gini Index'</span><span class="p">,</span> <span class="n">fontsize</span><span class="o">=</span><span class="mi">15</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">show</span><span class="p">()</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">
<div class="prompt"></div>
<div class="output_png output_subarea">
<img src="data:image/png;base64,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
"/>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="Decision-Tree-구현">Decision Tree 구현</h2>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="kn">from</span> <span class="nn">sklearn.tree</span> <span class="k">import</span> <span class="n">DecisionTreeClassifier</span>
<span class="kn">from</span> <span class="nn">sklearn.datasets</span> <span class="k">import</span> <span class="n">load_breast_cancer</span>
<span class="kn">from</span> <span class="nn">sklearn.model_selection</span> <span class="k">import</span> <span class="n">train_test_split</span>
<span class="kn">from</span> <span class="nn">sklearn.metrics</span> <span class="k">import</span> <span class="n">accuracy_score</span>

<span class="n">SEED</span> <span class="o">=</span> <span class="mi">42</span>
</pre></div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="c1"># breast cancer 데이터셋 로드</span>
<span class="n">cancer</span> <span class="o">=</span> <span class="n">load_breast_cancer</span><span class="p">()</span>
</pre></div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="c1"># train, test 데이터 분할</span>
<span class="n">x_train</span><span class="p">,</span> <span class="n">x_test</span><span class="p">,</span> <span class="n">y_train</span><span class="p">,</span> <span class="n">y_test</span> <span class="o">=</span> <span class="n">train_test_split</span><span class="p">(</span><span class="n">cancer</span><span class="p">[</span><span class="s1">'data'</span><span class="p">],</span> <span class="n">cancer</span><span class="p">[</span><span class="s1">'target'</span><span class="p">],</span> <span class="n">stratify</span><span class="o">=</span><span class="n">cancer</span><span class="p">[</span><span class="s1">'target'</span><span class="p">],</span> <span class="n">random_state</span><span class="o">=</span><span class="n">SEED</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="c1"># 알고리즘 정의</span>
<span class="n">tree</span> <span class="o">=</span> <span class="n">DecisionTreeClassifier</span><span class="p">(</span><span class="n">random_state</span><span class="o">=</span><span class="mi">0</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="c1"># 학습</span>
<span class="n">tree</span><span class="o">.</span><span class="n">fit</span><span class="p">(</span><span class="n">x_train</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">

<div class="output_text output_subarea output_execute_result">
<pre>DecisionTreeClassifier(random_state=0)</pre>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="c1"># 예측</span>
<span class="n">pred</span> <span class="o">=</span> <span class="n">tree</span><span class="o">.</span><span class="n">predict</span><span class="p">(</span><span class="n">x_test</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="c1"># 정확도 측정</span>
<span class="n">accuracy</span> <span class="o">=</span> <span class="n">accuracy_score</span><span class="p">(</span><span class="n">pred</span><span class="p">,</span> <span class="n">y_test</span><span class="p">)</span>
<span class="nb">print</span><span class="p">(</span><span class="n">f</span><span class="s1">'Accuracy Score: </span><span class="si">{accuracy:.3f}</span><span class="s1">'</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">
<div class="prompt"></div>
<div class="output_subarea output_stream output_stdout output_text">
<pre>Accuracy Score: 0.937
</pre>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="의사결정나무-시각화">의사결정나무 시각화</h2>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="kn">from</span> <span class="nn">sklearn.tree</span> <span class="k">import</span> <span class="n">export_graphviz</span>
<span class="kn">from</span> <span class="nn">sklearn.metrics</span> <span class="k">import</span> <span class="n">accuracy_score</span>
<span class="kn">import</span> <span class="nn">graphviz</span>

<span class="k">def</span> <span class="nf">show_trees</span><span class="p">(</span><span class="n">tree</span><span class="p">):</span>
    <span class="n">export_graphviz</span><span class="p">(</span><span class="n">tree</span><span class="p">,</span> <span class="n">out_file</span><span class="o">=</span><span class="s2">"tree.dot"</span><span class="p">,</span> <span class="n">class_names</span><span class="o">=</span><span class="p">[</span><span class="s2">"악성"</span><span class="p">,</span> <span class="s2">"양성"</span><span class="p">],</span>
                    <span class="n">feature_names</span><span class="o">=</span><span class="n">cancer</span><span class="p">[</span><span class="s1">'feature_names'</span><span class="p">],</span> 
                    <span class="n">precision</span><span class="o">=</span><span class="mi">3</span><span class="p">,</span> <span class="n">filled</span><span class="o">=</span><span class="kc">True</span><span class="p">)</span>
    <span class="k">with</span> <span class="nb">open</span><span class="p">(</span><span class="s2">"tree.dot"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="n">dot_graph</span> <span class="o">=</span> <span class="n">f</span><span class="o">.</span><span class="n">read</span><span class="p">()</span>
    <span class="n">pred</span> <span class="o">=</span> <span class="n">tree</span><span class="o">.</span><span class="n">predict</span><span class="p">(</span><span class="n">x_test</span><span class="p">)</span>
    <span class="nb">print</span><span class="p">(</span><span class="s1">'정확도: </span><span class="si">{:.2f}</span><span class="s1"> %'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">accuracy_score</span><span class="p">(</span><span class="n">y_test</span><span class="p">,</span> <span class="n">pred</span><span class="p">)</span> <span class="o">*</span> <span class="mi">100</span><span class="p">))</span>

    <span class="n">display</span><span class="p">(</span><span class="n">graphviz</span><span class="o">.</span><span class="n">Source</span><span class="p">(</span><span class="n">dot_graph</span><span class="p">))</span>
</pre></div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">show_trees</span><span class="p">(</span><span class="n">tree</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">
<div class="prompt"></div>
<div class="output_subarea output_stream output_stdout output_text">
<pre>정확도: 93.71 %
</pre>
</div>
</div>
<div class="output_area">
<div class="prompt"></div>
<div class="output_svg output_subarea">
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
 "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">

<!-- Generated by graphviz version 2.40.1 (20161225.0304)
 -->
<!-- Title: Tree Pages: 1 -->
<svg height="909pt" viewbox="0.00 0.00 1733.00 909.00" width="1733pt" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<g class="graph" id="graph0" transform="scale(1 1) rotate(0) translate(4 905)">
<title>Tree</title>
<polygon fill="#ffffff" points="-4,4 -4,-905 1729,-905 1729,4 -4,4" stroke="transparent"></polygon>
<!-- 0 -->
<g class="node" id="node1">
<title>0</title>
<polygon fill="#afd7f4" points="1245,-901 1056,-901 1056,-818 1245,-818 1245,-901" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1150.5" y="-885.8">worst radius &lt;= 16.795</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1150.5" y="-870.8">gini = 0.468</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1150.5" y="-855.8">samples = 426</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1150.5" y="-840.8">value = [159, 267]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1150.5" y="-825.8">class = 양성</text>
</g>
<!-- 1 -->
<g class="node" id="node2">
<title>1</title>
<polygon fill="#4ca6e8" points="1085.5,-782 843.5,-782 843.5,-699 1085.5,-699 1085.5,-782" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="964.5" y="-766.8">worst concave points &lt;= 0.136</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="964.5" y="-751.8">gini = 0.161</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="964.5" y="-736.8">samples = 284</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="964.5" y="-721.8">value = [25, 259]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="964.5" y="-706.8">class = 양성</text>
</g>
<!-- 0&#45;&gt;1 -->
<g class="edge" id="edge1">
<title>0-&gt;1</title>
<path d="M1085.4462,-817.8796C1070.1546,-808.0962 1053.7516,-797.6019 1038.114,-787.5971" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1039.8268,-784.538 1029.517,-782.0969 1036.0543,-790.4345 1039.8268,-784.538" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1034.881" y="-802.8147">True</text>
</g>
<!-- 28 -->
<g class="node" id="node29">
<title>28</title>
<polygon fill="#e78945" points="1322.5,-782 1138.5,-782 1138.5,-699 1322.5,-699 1322.5,-782" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1230.5" y="-766.8">texture error &lt;= 0.473</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1230.5" y="-751.8">gini = 0.106</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1230.5" y="-736.8">samples = 142</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1230.5" y="-721.8">value = [134, 8]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1230.5" y="-706.8">class = 악성</text>
</g>
<!-- 0&#45;&gt;28 -->
<g class="edge" id="edge28">
<title>0-&gt;28</title>
<path d="M1178.4801,-817.8796C1184.3531,-809.1434 1190.6073,-799.8404 1196.6679,-790.8253" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1199.7259,-792.5498 1202.4005,-782.2981 1193.9166,-788.6444 1199.7259,-792.5498" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1207.2085" y="-803.1314">False</text>
</g>
<!-- 2 -->
<g class="node" id="node3">
<title>2</title>
<polygon fill="#3c9fe5" points="765.5,-663 589.5,-663 589.5,-580 765.5,-580 765.5,-663" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="677.5" y="-647.8">radius error &lt;= 1.048</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="677.5" y="-632.8">gini = 0.031</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="677.5" y="-617.8">samples = 252</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="677.5" y="-602.8">value = [4, 248]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="677.5" y="-587.8">class = 양성</text>
</g>
<!-- 1&#45;&gt;2 -->
<g class="edge" id="edge2">
<title>1-&gt;2</title>
<path d="M864.1213,-698.8796C835.3661,-686.9567 804.0642,-673.9778 775.4565,-662.1161" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="776.4436,-658.7365 765.8656,-658.1394 773.7625,-665.2027 776.4436,-658.7365" stroke="#000000"></polygon>
</g>
<!-- 17 -->
<g class="node" id="node18">
<title>17</title>
<polygon fill="#f3c3a1" points="1058.5,-663 870.5,-663 870.5,-580 1058.5,-580 1058.5,-663" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="964.5" y="-647.8">worst texture &lt;= 25.62</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="964.5" y="-632.8">gini = 0.451</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="964.5" y="-617.8">samples = 32</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="964.5" y="-602.8">value = [21, 11]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="964.5" y="-587.8">class = 악성</text>
</g>
<!-- 1&#45;&gt;17 -->
<g class="edge" id="edge17">
<title>1-&gt;17</title>
<path d="M964.5,-698.8796C964.5,-690.6838 964.5,-681.9891 964.5,-673.5013" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="968.0001,-673.298 964.5,-663.2981 961.0001,-673.2981 968.0001,-673.298" stroke="#000000"></polygon>
</g>
<!-- 3 -->
<g class="node" id="node4">
<title>3</title>
<polygon fill="#3b9ee5" points="522.5,-544 306.5,-544 306.5,-461 522.5,-461 522.5,-544" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="414.5" y="-528.8">smoothness error &lt;= 0.003</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="414.5" y="-513.8">gini = 0.024</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="414.5" y="-498.8">samples = 251</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="414.5" y="-483.8">value = [3, 248]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="414.5" y="-468.8">class = 양성</text>
</g>
<!-- 2&#45;&gt;3 -->
<g class="edge" id="edge3">
<title>2-&gt;3</title>
<path d="M589.1932,-581.5437C565.6398,-570.8864 540.0066,-559.2882 515.8143,-548.3418" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="517.1739,-545.1154 506.6203,-544.1818 514.2882,-551.493 517.1739,-545.1154" stroke="#000000"></polygon>
</g>
<!-- 16 -->
<g class="node" id="node17">
<title>16</title>
<polygon fill="#e58139" points="734,-536.5 621,-536.5 621,-468.5 734,-468.5 734,-536.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="677.5" y="-521.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="677.5" y="-506.3">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="677.5" y="-491.3">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="677.5" y="-476.3">class = 악성</text>
</g>
<!-- 2&#45;&gt;16 -->
<g class="edge" id="edge16">
<title>2-&gt;16</title>
<path d="M677.5,-579.8796C677.5,-569.2134 677.5,-557.7021 677.5,-546.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="681.0001,-546.8149 677.5,-536.8149 674.0001,-546.815 681.0001,-546.8149" stroke="#000000"></polygon>
</g>
<!-- 4 -->
<g class="node" id="node5">
<title>4</title>
<polygon fill="#7bbeee" points="276,-425 99,-425 99,-342 276,-342 276,-425" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-409.8">mean texture &lt;= 19.9</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-394.8">gini = 0.375</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-379.8">samples = 4</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-364.8">value = [1, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-349.8">class = 양성</text>
</g>
<!-- 3&#45;&gt;4 -->
<g class="edge" id="edge4">
<title>3-&gt;4</title>
<path d="M335.1064,-460.8796C315.9256,-450.8244 295.3122,-440.0183 275.7525,-429.7645" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="277.3306,-426.6401 266.8488,-425.0969 274.0805,-432.8398 277.3306,-426.6401" stroke="#000000"></polygon>
</g>
<!-- 7 -->
<g class="node" id="node8">
<title>7</title>
<polygon fill="#3b9ee5" points="491.5,-425 337.5,-425 337.5,-342 491.5,-342 491.5,-425" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="414.5" y="-409.8">area error &lt;= 48.7</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="414.5" y="-394.8">gini = 0.016</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="414.5" y="-379.8">samples = 247</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="414.5" y="-364.8">value = [2, 245]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="414.5" y="-349.8">class = 양성</text>
</g>
<!-- 3&#45;&gt;7 -->
<g class="edge" id="edge7">
<title>3-&gt;7</title>
<path d="M414.5,-460.8796C414.5,-452.6838 414.5,-443.9891 414.5,-435.5013" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="418.0001,-435.298 414.5,-425.2981 411.0001,-435.2981 418.0001,-435.298" stroke="#000000"></polygon>
</g>
<!-- 5 -->
<g class="node" id="node6">
<title>5</title>
<polygon fill="#399de5" points="113,-298.5 0,-298.5 0,-230.5 113,-230.5 113,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="56.5" y="-283.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="56.5" y="-268.3">samples = 3</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="56.5" y="-253.3">value = [0, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="56.5" y="-238.3">class = 양성</text>
</g>
<!-- 4&#45;&gt;5 -->
<g class="edge" id="edge5">
<title>4-&gt;5</title>
<path d="M141.6826,-341.8796C128.7303,-330.1138 114.646,-317.3197 101.7286,-305.5855" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="104.0306,-302.9482 94.2753,-298.8149 99.3239,-308.1296 104.0306,-302.9482" stroke="#000000"></polygon>
</g>
<!-- 6 -->
<g class="node" id="node7">
<title>6</title>
<polygon fill="#e58139" points="244,-298.5 131,-298.5 131,-230.5 244,-230.5 244,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-283.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-268.3">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-253.3">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-238.3">class = 악성</text>
</g>
<!-- 4&#45;&gt;6 -->
<g class="edge" id="edge6">
<title>4-&gt;6</title>
<path d="M187.5,-341.8796C187.5,-331.2134 187.5,-319.7021 187.5,-308.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="191.0001,-308.8149 187.5,-298.8149 184.0001,-308.815 191.0001,-308.8149" stroke="#000000"></polygon>
</g>
<!-- 8 -->
<g class="node" id="node9">
<title>8</title>
<polygon fill="#3a9de5" points="450.5,-306 262.5,-306 262.5,-223 450.5,-223 450.5,-306" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="356.5" y="-290.8">worst texture &lt;= 33.35</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="356.5" y="-275.8">gini = 0.008</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="356.5" y="-260.8">samples = 243</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="356.5" y="-245.8">value = [1, 242]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="356.5" y="-230.8">class = 양성</text>
</g>
<!-- 7&#45;&gt;8 -->
<g class="edge" id="edge8">
<title>7-&gt;8</title>
<path d="M394.2144,-341.8796C390.0443,-333.3236 385.6091,-324.2238 381.3003,-315.3833" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="384.3997,-313.7538 376.8722,-306.2981 378.1073,-316.8207 384.3997,-313.7538" stroke="#000000"></polygon>
</g>
<!-- 13 -->
<g class="node" id="node14">
<title>13</title>
<polygon fill="#7bbeee" points="672,-306 469,-306 469,-223 672,-223 672,-306" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="570.5" y="-290.8">mean concavity &lt;= 0.029</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="570.5" y="-275.8">gini = 0.375</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="570.5" y="-260.8">samples = 4</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="570.5" y="-245.8">value = [1, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="570.5" y="-230.8">class = 양성</text>
</g>
<!-- 7&#45;&gt;13 -->
<g class="edge" id="edge13">
<title>7-&gt;13</title>
<path d="M469.0612,-341.8796C481.5302,-332.368 494.8802,-322.1843 507.6648,-312.432" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="510.1415,-314.9448 515.9696,-306.0969 505.896,-309.3792 510.1415,-314.9448" stroke="#000000"></polygon>
</g>
<!-- 9 -->
<g class="node" id="node10">
<title>9</title>
<polygon fill="#399de5" points="262,-179.5 131,-179.5 131,-111.5 262,-111.5 262,-179.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="196.5" y="-164.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="196.5" y="-149.3">samples = 225</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="196.5" y="-134.3">value = [0, 225]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="196.5" y="-119.3">class = 양성</text>
</g>
<!-- 8&#45;&gt;9 -->
<g class="edge" id="edge9">
<title>8-&gt;9</title>
<path d="M300.5398,-222.8796C284.3478,-210.8368 266.7073,-197.7167 250.6292,-185.7586" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="252.4925,-182.7825 242.3797,-179.623 248.315,-188.3993 252.4925,-182.7825" stroke="#000000"></polygon>
</g>
<!-- 10 -->
<g class="node" id="node11">
<title>10</title>
<polygon fill="#45a3e7" points="459,-187 280,-187 280,-104 459,-104 459,-187" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="369.5" y="-171.8">worst texture &lt;= 33.8</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="369.5" y="-156.8">gini = 0.105</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="369.5" y="-141.8">samples = 18</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="369.5" y="-126.8">value = [1, 17]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="369.5" y="-111.8">class = 양성</text>
</g>
<!-- 8&#45;&gt;10 -->
<g class="edge" id="edge10">
<title>8-&gt;10</title>
<path d="M361.0468,-222.8796C361.9421,-214.6838 362.8919,-205.9891 363.8192,-197.5013" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="367.3271,-197.6191 364.9338,-187.2981 360.3685,-196.8588 367.3271,-197.6191" stroke="#000000"></polygon>
</g>
<!-- 11 -->
<g class="node" id="node12">
<title>11</title>
<polygon fill="#e58139" points="358,-68 245,-68 245,0 358,0 358,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="301.5" y="-52.8">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="301.5" y="-37.8">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="301.5" y="-22.8">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="301.5" y="-7.8">class = 악성</text>
</g>
<!-- 10&#45;&gt;11 -->
<g class="edge" id="edge11">
<title>10-&gt;11</title>
<path d="M344.1793,-103.9815C338.7984,-95.1585 333.1068,-85.8258 327.6941,-76.9506" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="330.5912,-74.9789 322.3962,-68.2637 324.6149,-78.6236 330.5912,-74.9789" stroke="#000000"></polygon>
</g>
<!-- 12 -->
<g class="node" id="node13">
<title>12</title>
<polygon fill="#399de5" points="498.5,-68 376.5,-68 376.5,0 498.5,0 498.5,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="437.5" y="-52.8">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="437.5" y="-37.8">samples = 17</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="437.5" y="-22.8">value = [0, 17]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="437.5" y="-7.8">class = 양성</text>
</g>
<!-- 10&#45;&gt;12 -->
<g class="edge" id="edge12">
<title>10-&gt;12</title>
<path d="M394.8207,-103.9815C400.2016,-95.1585 405.8932,-85.8258 411.3059,-76.9506" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="414.3851,-78.6236 416.6038,-68.2637 408.4088,-74.9789 414.3851,-78.6236" stroke="#000000"></polygon>
</g>
<!-- 14 -->
<g class="node" id="node15">
<title>14</title>
<polygon fill="#e58139" points="615,-179.5 502,-179.5 502,-111.5 615,-111.5 615,-179.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="558.5" y="-164.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="558.5" y="-149.3">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="558.5" y="-134.3">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="558.5" y="-119.3">class = 악성</text>
</g>
<!-- 13&#45;&gt;14 -->
<g class="edge" id="edge14">
<title>13-&gt;14</title>
<path d="M566.303,-222.8796C565.2274,-212.2134 564.0666,-200.7021 562.9775,-189.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="566.4461,-189.4133 561.9603,-179.8149 559.4814,-190.1157 566.4461,-189.4133" stroke="#000000"></polygon>
</g>
<!-- 15 -->
<g class="node" id="node16">
<title>15</title>
<polygon fill="#399de5" points="746,-179.5 633,-179.5 633,-111.5 746,-111.5 746,-179.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="689.5" y="-164.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="689.5" y="-149.3">samples = 3</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="689.5" y="-134.3">value = [0, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="689.5" y="-119.3">class = 양성</text>
</g>
<!-- 13&#45;&gt;15 -->
<g class="edge" id="edge15">
<title>13-&gt;15</title>
<path d="M612.1204,-222.8796C623.7763,-211.2237 636.4413,-198.5587 648.0852,-186.9148" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="650.5889,-189.3609 655.1851,-179.8149 645.6391,-184.4111 650.5889,-189.3609" stroke="#000000"></polygon>
</g>
<!-- 18 -->
<g class="node" id="node19">
<title>18</title>
<polygon fill="#7bbeee" points="1013,-544 846,-544 846,-461 1013,-461 1013,-544" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="929.5" y="-528.8">worst area &lt;= 817.1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="929.5" y="-513.8">gini = 0.375</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="929.5" y="-498.8">samples = 12</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="929.5" y="-483.8">value = [3, 9]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="929.5" y="-468.8">class = 양성</text>
</g>
<!-- 17&#45;&gt;18 -->
<g class="edge" id="edge18">
<title>17-&gt;18</title>
<path d="M952.2587,-579.8796C949.7952,-571.5037 947.1784,-562.6067 944.63,-553.942" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="947.9731,-552.9041 941.7936,-544.2981 941.2575,-554.8793 947.9731,-552.9041" stroke="#000000"></polygon>
</g>
<!-- 23 -->
<g class="node" id="node24">
<title>23</title>
<polygon fill="#e88f4f" points="1238,-544 1031,-544 1031,-461 1238,-461 1238,-544" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1134.5" y="-528.8">worst symmetry &lt;= 0.268</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1134.5" y="-513.8">gini = 0.18</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1134.5" y="-498.8">samples = 20</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1134.5" y="-483.8">value = [18, 2]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1134.5" y="-468.8">class = 악성</text>
</g>
<!-- 17&#45;&gt;23 -->
<g class="edge" id="edge23">
<title>17-&gt;23</title>
<path d="M1023.9578,-579.8796C1037.8046,-570.1868 1052.6484,-559.7961 1066.8212,-549.8752" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1068.8906,-552.6989 1075.0758,-544.0969 1064.8764,-546.9643 1068.8906,-552.6989" stroke="#000000"></polygon>
</g>
<!-- 19 -->
<g class="node" id="node20">
<title>19</title>
<polygon fill="#4fa8e8" points="855.5,-425 637.5,-425 637.5,-342 855.5,-342 855.5,-425" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="746.5" y="-409.8">mean smoothness &lt;= 0.123</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="746.5" y="-394.8">gini = 0.18</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="746.5" y="-379.8">samples = 10</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="746.5" y="-364.8">value = [1, 9]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="746.5" y="-349.8">class = 양성</text>
</g>
<!-- 18&#45;&gt;19 -->
<g class="edge" id="edge19">
<title>18-&gt;19</title>
<path d="M865.4955,-460.8796C850.4505,-451.0962 834.3121,-440.6019 818.9267,-430.5971" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="820.7598,-427.6143 810.4684,-425.0969 816.9437,-433.4827 820.7598,-427.6143" stroke="#000000"></polygon>
</g>
<!-- 22 -->
<g class="node" id="node23">
<title>22</title>
<polygon fill="#e58139" points="987,-417.5 874,-417.5 874,-349.5 987,-349.5 987,-417.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="930.5" y="-402.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="930.5" y="-387.3">samples = 2</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="930.5" y="-372.3">value = [2, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="930.5" y="-357.3">class = 악성</text>
</g>
<!-- 18&#45;&gt;22 -->
<g class="edge" id="edge22">
<title>18-&gt;22</title>
<path d="M929.8498,-460.8796C929.9394,-450.2134 930.0361,-438.7021 930.1269,-427.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="933.6274,-427.844 930.2116,-417.8149 926.6276,-427.7851 933.6274,-427.844" stroke="#000000"></polygon>
</g>
<!-- 20 -->
<g class="node" id="node21">
<title>20</title>
<polygon fill="#399de5" points="803,-298.5 690,-298.5 690,-230.5 803,-230.5 803,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="746.5" y="-283.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="746.5" y="-268.3">samples = 9</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="746.5" y="-253.3">value = [0, 9]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="746.5" y="-238.3">class = 양성</text>
</g>
<!-- 19&#45;&gt;20 -->
<g class="edge" id="edge20">
<title>19-&gt;20</title>
<path d="M746.5,-341.8796C746.5,-331.2134 746.5,-319.7021 746.5,-308.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="750.0001,-308.8149 746.5,-298.8149 743.0001,-308.815 750.0001,-308.8149" stroke="#000000"></polygon>
</g>
<!-- 21 -->
<g class="node" id="node22">
<title>21</title>
<polygon fill="#e58139" points="934,-298.5 821,-298.5 821,-230.5 934,-230.5 934,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="877.5" y="-283.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="877.5" y="-268.3">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="877.5" y="-253.3">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="877.5" y="-238.3">class = 악성</text>
</g>
<!-- 19&#45;&gt;21 -->
<g class="edge" id="edge21">
<title>19-&gt;21</title>
<path d="M792.3174,-341.8796C805.2697,-330.1138 819.354,-317.3197 832.2714,-305.5855" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="834.6761,-308.1296 839.7247,-298.8149 829.9694,-302.9482 834.6761,-308.1296" stroke="#000000"></polygon>
</g>
<!-- 24 -->
<g class="node" id="node25">
<title>24</title>
<polygon fill="#9ccef2" points="1262,-425 1005,-425 1005,-342 1262,-342 1262,-425" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1133.5" y="-409.8">fractal dimension error &lt;= 0.002</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1133.5" y="-394.8">gini = 0.444</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1133.5" y="-379.8">samples = 3</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1133.5" y="-364.8">value = [1, 2]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1133.5" y="-349.8">class = 양성</text>
</g>
<!-- 23&#45;&gt;24 -->
<g class="edge" id="edge24">
<title>23-&gt;24</title>
<path d="M1134.1502,-460.8796C1134.0814,-452.6838 1134.0083,-443.9891 1133.937,-435.5013" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1137.4352,-435.2683 1133.8512,-425.2981 1130.4355,-435.3272 1137.4352,-435.2683" stroke="#000000"></polygon>
</g>
<!-- 27 -->
<g class="node" id="node28">
<title>27</title>
<polygon fill="#e58139" points="1402.5,-417.5 1280.5,-417.5 1280.5,-349.5 1402.5,-349.5 1402.5,-417.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1341.5" y="-402.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1341.5" y="-387.3">samples = 17</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1341.5" y="-372.3">value = [17, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1341.5" y="-357.3">class = 악성</text>
</g>
<!-- 23&#45;&gt;27 -->
<g class="edge" id="edge27">
<title>23-&gt;27</title>
<path d="M1207.9892,-460.8176C1228.2684,-449.2621 1250.2583,-436.6826 1270.5,-425 1271.8139,-424.2417 1273.141,-423.4748 1274.478,-422.7015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1276.5135,-425.5671 1283.4097,-417.5239 1273.0029,-419.511 1276.5135,-425.5671" stroke="#000000"></polygon>
</g>
<!-- 25 -->
<g class="node" id="node26">
<title>25</title>
<polygon fill="#e58139" points="1128,-298.5 1015,-298.5 1015,-230.5 1128,-230.5 1128,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1071.5" y="-283.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1071.5" y="-268.3">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1071.5" y="-253.3">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1071.5" y="-238.3">class = 악성</text>
</g>
<!-- 24&#45;&gt;25 -->
<g class="edge" id="edge25">
<title>24-&gt;25</title>
<path d="M1111.8154,-341.8796C1106.0864,-330.8835 1099.8894,-318.9893 1094.1126,-307.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1097.103,-306.0662 1089.3784,-298.8149 1090.895,-309.3007 1097.103,-306.0662" stroke="#000000"></polygon>
</g>
<!-- 26 -->
<g class="node" id="node27">
<title>26</title>
<polygon fill="#399de5" points="1259,-298.5 1146,-298.5 1146,-230.5 1259,-230.5 1259,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1202.5" y="-283.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1202.5" y="-268.3">samples = 2</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1202.5" y="-253.3">value = [0, 2]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1202.5" y="-238.3">class = 양성</text>
</g>
<!-- 24&#45;&gt;26 -->
<g class="edge" id="edge26">
<title>24-&gt;26</title>
<path d="M1157.6329,-341.8796C1164.0725,-330.7735 1171.0434,-318.7513 1177.5271,-307.5691" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1180.6148,-309.2215 1182.6031,-298.8149 1174.5591,-305.7102 1180.6148,-309.2215" stroke="#000000"></polygon>
</g>
<!-- 29 -->
<g class="node" id="node30">
<title>29</title>
<polygon fill="#399de5" points="1287,-655.5 1174,-655.5 1174,-587.5 1287,-587.5 1287,-655.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1230.5" y="-640.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1230.5" y="-625.3">samples = 5</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1230.5" y="-610.3">value = [0, 5]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1230.5" y="-595.3">class = 양성</text>
</g>
<!-- 28&#45;&gt;29 -->
<g class="edge" id="edge29">
<title>28-&gt;29</title>
<path d="M1230.5,-698.8796C1230.5,-688.2134 1230.5,-676.7021 1230.5,-665.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1234.0001,-665.8149 1230.5,-655.8149 1227.0001,-665.815 1234.0001,-665.8149" stroke="#000000"></polygon>
</g>
<!-- 30 -->
<g class="node" id="node31">
<title>30</title>
<polygon fill="#e6843d" points="1579.5,-663 1375.5,-663 1375.5,-580 1579.5,-580 1579.5,-663" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1477.5" y="-647.8">worst concavity &lt;= 0.191</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1477.5" y="-632.8">gini = 0.043</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1477.5" y="-617.8">samples = 137</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1477.5" y="-602.8">value = [134, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1477.5" y="-587.8">class = 악성</text>
</g>
<!-- 28&#45;&gt;30 -->
<g class="edge" id="edge30">
<title>28-&gt;30</title>
<path d="M1316.8886,-698.8796C1337.9474,-688.7339 1360.5932,-677.8235 1382.0468,-667.4876" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1383.6703,-670.5905 1391.1602,-663.0969 1380.6321,-664.2842 1383.6703,-670.5905" stroke="#000000"></polygon>
</g>
<!-- 31 -->
<g class="node" id="node32">
<title>31</title>
<polygon fill="#bddef6" points="1576,-544 1379,-544 1379,-461 1576,-461 1576,-544" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1477.5" y="-528.8">worst texture &lt;= 30.975</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1477.5" y="-513.8">gini = 0.48</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1477.5" y="-498.8">samples = 5</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1477.5" y="-483.8">value = [2, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1477.5" y="-468.8">class = 양성</text>
</g>
<!-- 30&#45;&gt;31 -->
<g class="edge" id="edge31">
<title>30-&gt;31</title>
<path d="M1477.5,-579.8796C1477.5,-571.6838 1477.5,-562.9891 1477.5,-554.5013" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1481.0001,-554.298 1477.5,-544.2981 1474.0001,-554.2981 1481.0001,-554.298" stroke="#000000"></polygon>
</g>
<!-- 34 -->
<g class="node" id="node35">
<title>34</title>
<polygon fill="#e58139" points="1725,-536.5 1594,-536.5 1594,-468.5 1725,-468.5 1725,-536.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1659.5" y="-521.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1659.5" y="-506.3">samples = 132</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1659.5" y="-491.3">value = [132, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1659.5" y="-476.3">class = 악성</text>
</g>
<!-- 30&#45;&gt;34 -->
<g class="edge" id="edge34">
<title>30-&gt;34</title>
<path d="M1541.1548,-579.8796C1559.9111,-567.6158 1580.3761,-554.2348 1598.933,-542.1015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1600.8575,-545.025 1607.3118,-536.623 1597.0267,-539.1662 1600.8575,-545.025" stroke="#000000"></polygon>
</g>
<!-- 32 -->
<g class="node" id="node33">
<title>32</title>
<polygon fill="#399de5" points="1534,-417.5 1421,-417.5 1421,-349.5 1534,-349.5 1534,-417.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1477.5" y="-402.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1477.5" y="-387.3">samples = 3</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1477.5" y="-372.3">value = [0, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1477.5" y="-357.3">class = 양성</text>
</g>
<!-- 31&#45;&gt;32 -->
<g class="edge" id="edge32">
<title>31-&gt;32</title>
<path d="M1477.5,-460.8796C1477.5,-450.2134 1477.5,-438.7021 1477.5,-427.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1481.0001,-427.8149 1477.5,-417.8149 1474.0001,-427.815 1481.0001,-427.8149" stroke="#000000"></polygon>
</g>
<!-- 33 -->
<g class="node" id="node34">
<title>33</title>
<polygon fill="#e58139" points="1665,-417.5 1552,-417.5 1552,-349.5 1665,-349.5 1665,-417.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1608.5" y="-402.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1608.5" y="-387.3">samples = 2</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1608.5" y="-372.3">value = [2, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1608.5" y="-357.3">class = 악성</text>
</g>
<!-- 31&#45;&gt;33 -->
<g class="edge" id="edge33">
<title>31-&gt;33</title>
<path d="M1523.3174,-460.8796C1536.2697,-449.1138 1550.354,-436.3197 1563.2714,-424.5855" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1565.6761,-427.1296 1570.7247,-417.8149 1560.9694,-421.9482 1565.6761,-427.1296" stroke="#000000"></polygon>
</g>
</g>
</svg>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="주요-Hyper-Parameter">주요 Hyper Parameter</h2>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h3 id="max_depth">max_depth</h3>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><code>max_depth</code>는 최대 트리의 깊이를 제한 합니다.</p>
<p>기본 값은 None, 제한 없음 입니다.</p>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">tree</span> <span class="o">=</span> <span class="n">DecisionTreeClassifier</span><span class="p">(</span><span class="n">max_depth</span><span class="o">=</span><span class="mi">3</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="n">SEED</span><span class="p">)</span>
<span class="n">tree</span><span class="o">.</span><span class="n">fit</span><span class="p">(</span><span class="n">x_train</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>
<span class="n">show_trees</span><span class="p">(</span><span class="n">tree</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">
<div class="prompt"></div>
<div class="output_subarea output_stream output_stdout output_text">
<pre>정확도: 94.41 %
</pre>
</div>
</div>
<div class="output_area">
<div class="prompt"></div>
<div class="output_svg output_subarea">
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
 "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">

<!-- Generated by graphviz version 2.40.1 (20161225.0304)
 -->
<!-- Title: Tree Pages: 1 -->
<svg height="433pt" viewbox="0.00 0.00 917.00 433.00" width="917pt" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<g class="graph" id="graph0" transform="scale(1 1) rotate(0) translate(4 429)">
<title>Tree</title>
<polygon fill="#ffffff" points="-4,4 -4,-429 913,-429 913,4 -4,4" stroke="transparent"></polygon>
<!-- 0 -->
<g class="node" id="node1">
<title>0</title>
<polygon fill="#afd7f4" points="580,-425 391,-425 391,-342 580,-342 580,-425" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="485.5" y="-409.8">worst radius &lt;= 16.795</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="485.5" y="-394.8">gini = 0.468</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="485.5" y="-379.8">samples = 426</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="485.5" y="-364.8">value = [159, 267]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="485.5" y="-349.8">class = 양성</text>
</g>
<!-- 1 -->
<g class="node" id="node2">
<title>1</title>
<polygon fill="#4ca6e8" points="491.5,-306 249.5,-306 249.5,-223 491.5,-223 491.5,-306" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="370.5" y="-290.8">worst concave points &lt;= 0.136</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="370.5" y="-275.8">gini = 0.161</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="370.5" y="-260.8">samples = 284</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="370.5" y="-245.8">value = [25, 259]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="370.5" y="-230.8">class = 양성</text>
</g>
<!-- 0&#45;&gt;1 -->
<g class="edge" id="edge1">
<title>0-&gt;1</title>
<path d="M445.2786,-341.8796C436.488,-332.7832 427.1034,-323.0722 418.0574,-313.7116" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="420.3591,-311.0568 410.8931,-306.2981 415.3255,-315.9212 420.3591,-311.0568" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="410.4658" y="-327.5944">True</text>
</g>
<!-- 8 -->
<g class="node" id="node9">
<title>8</title>
<polygon fill="#e78945" points="693.5,-306 509.5,-306 509.5,-223 693.5,-223 693.5,-306" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="601.5" y="-290.8">texture error &lt;= 0.473</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="601.5" y="-275.8">gini = 0.106</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="601.5" y="-260.8">samples = 142</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="601.5" y="-245.8">value = [134, 8]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="601.5" y="-230.8">class = 악성</text>
</g>
<!-- 0&#45;&gt;8 -->
<g class="edge" id="edge8">
<title>0-&gt;8</title>
<path d="M526.0712,-341.8796C534.9382,-332.7832 544.4044,-323.0722 553.5291,-313.7116" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="556.2817,-315.9019 560.7557,-306.2981 551.2691,-311.0158 556.2817,-315.9019" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="561.0748" y="-327.596">False</text>
</g>
<!-- 2 -->
<g class="node" id="node3">
<title>2</title>
<polygon fill="#3c9fe5" points="274.5,-187 102.5,-187 102.5,-104 274.5,-104 274.5,-187" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="188.5" y="-171.8">area error &lt;= 91.555</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="188.5" y="-156.8">gini = 0.031</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="188.5" y="-141.8">samples = 252</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="188.5" y="-126.8">value = [4, 248]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="188.5" y="-111.8">class = 양성</text>
</g>
<!-- 1&#45;&gt;2 -->
<g class="edge" id="edge2">
<title>1-&gt;2</title>
<path d="M306.8452,-222.8796C291.8824,-213.0962 275.8323,-202.6019 260.5309,-192.5971" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="262.4039,-189.6401 252.1188,-187.0969 258.5731,-195.4989 262.4039,-189.6401" stroke="#000000"></polygon>
</g>
<!-- 5 -->
<g class="node" id="node6">
<title>5</title>
<polygon fill="#f3c3a1" points="480.5,-187 292.5,-187 292.5,-104 480.5,-104 480.5,-187" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="386.5" y="-171.8">worst texture &lt;= 25.62</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="386.5" y="-156.8">gini = 0.451</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="386.5" y="-141.8">samples = 32</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="386.5" y="-126.8">value = [21, 11]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="386.5" y="-111.8">class = 악성</text>
</g>
<!-- 1&#45;&gt;5 -->
<g class="edge" id="edge5">
<title>1-&gt;5</title>
<path d="M376.096,-222.8796C377.2101,-214.5938 378.3927,-205.798 379.5458,-197.2216" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="383.0162,-197.6753 380.8801,-187.2981 376.0787,-196.7425 383.0162,-197.6753" stroke="#000000"></polygon>
</g>
<!-- 3 -->
<g class="node" id="node4">
<title>3</title>
<polygon fill="#3b9ee5" points="131,-68 0,-68 0,0 131,0 131,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="65.5" y="-52.8">gini = 0.024</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="65.5" y="-37.8">samples = 251</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="65.5" y="-22.8">value = [3, 248]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="65.5" y="-7.8">class = 양성</text>
</g>
<!-- 2&#45;&gt;3 -->
<g class="edge" id="edge3">
<title>2-&gt;3</title>
<path d="M142.6993,-103.9815C132.2566,-94.5151 121.1667,-84.462 110.7472,-75.0168" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="113.0572,-72.3868 103.2976,-68.2637 108.3559,-77.5731 113.0572,-72.3868" stroke="#000000"></polygon>
</g>
<!-- 4 -->
<g class="node" id="node5">
<title>4</title>
<polygon fill="#e58139" points="262,-68 149,-68 149,0 262,0 262,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="205.5" y="-52.8">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="205.5" y="-37.8">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="205.5" y="-22.8">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="205.5" y="-7.8">class = 악성</text>
</g>
<!-- 2&#45;&gt;4 -->
<g class="edge" id="edge4">
<title>2-&gt;4</title>
<path d="M194.8302,-103.9815C196.1053,-95.618 197.4503,-86.7965 198.7395,-78.3409" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="202.2286,-78.677 200.2759,-68.2637 195.3086,-77.6219 202.2286,-78.677" stroke="#000000"></polygon>
</g>
<!-- 6 -->
<g class="node" id="node7">
<title>6</title>
<polygon fill="#7bbeee" points="427.5,-68 313.5,-68 313.5,0 427.5,0 427.5,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="370.5" y="-52.8">gini = 0.375</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="370.5" y="-37.8">samples = 12</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="370.5" y="-22.8">value = [3, 9]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="370.5" y="-7.8">class = 양성</text>
</g>
<!-- 5&#45;&gt;6 -->
<g class="edge" id="edge6">
<title>5-&gt;6</title>
<path d="M380.5422,-103.9815C379.342,-95.618 378.0762,-86.7965 376.8628,-78.3409" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="380.3018,-77.6651 375.4168,-68.2637 373.3728,-78.6595 380.3018,-77.6651" stroke="#000000"></polygon>
</g>
<!-- 7 -->
<g class="node" id="node8">
<title>7</title>
<polygon fill="#e88f4f" points="567.5,-68 445.5,-68 445.5,0 567.5,0 567.5,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="506.5" y="-52.8">gini = 0.18</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="506.5" y="-37.8">samples = 20</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="506.5" y="-22.8">value = [18, 2]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="506.5" y="-7.8">class = 악성</text>
</g>
<!-- 5&#45;&gt;7 -->
<g class="edge" id="edge7">
<title>5-&gt;7</title>
<path d="M431.1836,-103.9815C441.2727,-94.607 451.981,-84.6572 462.0601,-75.2921" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="464.6809,-77.6346 469.6243,-68.2637 459.9161,-72.5065 464.6809,-77.6346" stroke="#000000"></polygon>
</g>
<!-- 9 -->
<g class="node" id="node10">
<title>9</title>
<polygon fill="#399de5" points="643,-179.5 530,-179.5 530,-111.5 643,-111.5 643,-179.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="586.5" y="-164.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="586.5" y="-149.3">samples = 5</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="586.5" y="-134.3">value = [0, 5]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="586.5" y="-119.3">class = 양성</text>
</g>
<!-- 8&#45;&gt;9 -->
<g class="edge" id="edge9">
<title>8-&gt;9</title>
<path d="M596.2537,-222.8796C594.9092,-212.2134 593.4582,-200.7021 592.0968,-189.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="595.5486,-189.2987 590.8254,-179.8149 588.6036,-190.1742 595.5486,-189.2987" stroke="#000000"></polygon>
</g>
<!-- 10 -->
<g class="node" id="node11">
<title>10</title>
<polygon fill="#e6843d" points="865.5,-187 661.5,-187 661.5,-104 865.5,-104 865.5,-187" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="763.5" y="-171.8">worst concavity &lt;= 0.191</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="763.5" y="-156.8">gini = 0.043</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="763.5" y="-141.8">samples = 137</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="763.5" y="-126.8">value = [134, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="763.5" y="-111.8">class = 악성</text>
</g>
<!-- 8&#45;&gt;10 -->
<g class="edge" id="edge10">
<title>8-&gt;10</title>
<path d="M658.1597,-222.8796C671.2316,-213.2774 685.2359,-202.9903 698.6273,-193.1534" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="700.885,-195.8378 706.8723,-187.0969 696.7409,-190.1963 700.885,-195.8378" stroke="#000000"></polygon>
</g>
<!-- 11 -->
<g class="node" id="node12">
<title>11</title>
<polygon fill="#bddef6" points="760,-68 647,-68 647,0 760,0 760,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="703.5" y="-52.8">gini = 0.48</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="703.5" y="-37.8">samples = 5</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="703.5" y="-22.8">value = [2, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="703.5" y="-7.8">class = 양성</text>
</g>
<!-- 10&#45;&gt;11 -->
<g class="edge" id="edge11">
<title>10-&gt;11</title>
<path d="M741.1582,-103.9815C736.4598,-95.2504 731.4929,-86.0202 726.7617,-77.2281" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="729.7586,-75.4111 721.9379,-68.2637 723.5944,-78.7282 729.7586,-75.4111" stroke="#000000"></polygon>
</g>
<!-- 12 -->
<g class="node" id="node13">
<title>12</title>
<polygon fill="#e58139" points="909,-68 778,-68 778,0 909,0 909,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="843.5" y="-52.8">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="843.5" y="-37.8">samples = 132</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="843.5" y="-22.8">value = [132, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="843.5" y="-7.8">class = 악성</text>
</g>
<!-- 10&#45;&gt;12 -->
<g class="edge" id="edge12">
<title>10-&gt;12</title>
<path d="M793.289,-103.9815C799.7514,-94.9747 806.5948,-85.4367 813.081,-76.3965" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="815.9303,-78.4291 818.9162,-68.2637 810.2428,-74.3483 815.9303,-78.4291" stroke="#000000"></polygon>
</g>
</g>
</svg>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h3 id="min_sample_split">min_sample_split</h3>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><code>min_sample_split</code>은 노드 내에서 분할이 필요한 최소의 샘플 숫자입니다.</p>
<p>기본 값은 2입니다.</p>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">tree</span> <span class="o">=</span> <span class="n">DecisionTreeClassifier</span><span class="p">(</span><span class="n">max_depth</span><span class="o">=</span><span class="mi">6</span><span class="p">,</span> <span class="n">min_samples_split</span><span class="o">=</span><span class="mi">20</span><span class="p">,</span>  <span class="n">random_state</span><span class="o">=</span><span class="n">SEED</span><span class="p">)</span>
<span class="n">tree</span><span class="o">.</span><span class="n">fit</span><span class="p">(</span><span class="n">x_train</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>
<span class="n">show_trees</span><span class="p">(</span><span class="n">tree</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">
<div class="prompt"></div>
<div class="output_subarea output_stream output_stdout output_text">
<pre>정확도: 94.41 %
</pre>
</div>
</div>
<div class="output_area">
<div class="prompt"></div>
<div class="output_svg output_subarea">
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
 "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">

<!-- Generated by graphviz version 2.40.1 (20161225.0304)
 -->
<!-- Title: Tree Pages: 1 -->
<svg height="790pt" viewbox="0.00 0.00 1144.00 790.00" width="1144pt" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<g class="graph" id="graph0" transform="scale(1 1) rotate(0) translate(4 786)">
<title>Tree</title>
<polygon fill="#ffffff" points="-4,4 -4,-786 1140,-786 1140,4 -4,4" stroke="transparent"></polygon>
<!-- 0 -->
<g class="node" id="node1">
<title>0</title>
<polygon fill="#afd7f4" points="763,-782 574,-782 574,-699 763,-699 763,-782" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="668.5" y="-766.8">worst radius &lt;= 16.795</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="668.5" y="-751.8">gini = 0.468</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="668.5" y="-736.8">samples = 426</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="668.5" y="-721.8">value = [159, 267]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="668.5" y="-706.8">class = 양성</text>
</g>
<!-- 1 -->
<g class="node" id="node2">
<title>1</title>
<polygon fill="#4ca6e8" points="674.5,-663 432.5,-663 432.5,-580 674.5,-580 674.5,-663" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="553.5" y="-647.8">worst concave points &lt;= 0.136</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="553.5" y="-632.8">gini = 0.161</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="553.5" y="-617.8">samples = 284</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="553.5" y="-602.8">value = [25, 259]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="553.5" y="-587.8">class = 양성</text>
</g>
<!-- 0&#45;&gt;1 -->
<g class="edge" id="edge1">
<title>0-&gt;1</title>
<path d="M628.2786,-698.8796C619.488,-689.7832 610.1034,-680.0722 601.0574,-670.7116" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="603.3591,-668.0568 593.8931,-663.2981 598.3255,-672.9212 603.3591,-668.0568" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="593.4658" y="-684.5944">True</text>
</g>
<!-- 16 -->
<g class="node" id="node17">
<title>16</title>
<polygon fill="#e78945" points="876.5,-663 692.5,-663 692.5,-580 876.5,-580 876.5,-663" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="784.5" y="-647.8">texture error &lt;= 0.473</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="784.5" y="-632.8">gini = 0.106</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="784.5" y="-617.8">samples = 142</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="784.5" y="-602.8">value = [134, 8]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="784.5" y="-587.8">class = 악성</text>
</g>
<!-- 0&#45;&gt;16 -->
<g class="edge" id="edge16">
<title>0-&gt;16</title>
<path d="M709.0712,-698.8796C717.9382,-689.7832 727.4044,-680.0722 736.5291,-670.7116" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="739.2817,-672.9019 743.7557,-663.2981 734.2691,-668.0158 739.2817,-672.9019" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="744.0748" y="-684.596">False</text>
</g>
<!-- 2 -->
<g class="node" id="node3">
<title>2</title>
<polygon fill="#3c9fe5" points="452.5,-544 280.5,-544 280.5,-461 452.5,-461 452.5,-544" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="366.5" y="-528.8">area error &lt;= 91.555</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="366.5" y="-513.8">gini = 0.031</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="366.5" y="-498.8">samples = 252</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="366.5" y="-483.8">value = [4, 248]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="366.5" y="-468.8">class = 양성</text>
</g>
<!-- 1&#45;&gt;2 -->
<g class="edge" id="edge2">
<title>1-&gt;2</title>
<path d="M488.0965,-579.8796C472.7226,-570.0962 456.2315,-559.6019 440.5098,-549.5971" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="442.1823,-546.5129 431.8666,-544.0969 438.4241,-552.4185 442.1823,-546.5129" stroke="#000000"></polygon>
</g>
<!-- 11 -->
<g class="node" id="node12">
<title>11</title>
<polygon fill="#f3c3a1" points="658.5,-544 470.5,-544 470.5,-461 658.5,-461 658.5,-544" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="564.5" y="-528.8">worst texture &lt;= 25.62</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="564.5" y="-513.8">gini = 0.451</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="564.5" y="-498.8">samples = 32</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="564.5" y="-483.8">value = [21, 11]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="564.5" y="-468.8">class = 악성</text>
</g>
<!-- 1&#45;&gt;11 -->
<g class="edge" id="edge11">
<title>1-&gt;11</title>
<path d="M557.3473,-579.8796C558.1049,-571.6838 558.9086,-562.9891 559.6932,-554.5013" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="563.2009,-554.5778 560.6363,-544.2981 556.2306,-553.9334 563.2009,-554.5778" stroke="#000000"></polygon>
</g>
<!-- 3 -->
<g class="node" id="node4">
<title>3</title>
<polygon fill="#3b9ee5" points="308.5,-425 154.5,-425 154.5,-342 308.5,-342 308.5,-425" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="231.5" y="-409.8">area error &lt;= 48.7</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="231.5" y="-394.8">gini = 0.024</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="231.5" y="-379.8">samples = 251</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="231.5" y="-364.8">value = [3, 248]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="231.5" y="-349.8">class = 양성</text>
</g>
<!-- 2&#45;&gt;3 -->
<g class="edge" id="edge3">
<title>2-&gt;3</title>
<path d="M319.2836,-460.8796C308.7598,-451.6031 297.5109,-441.6874 286.6979,-432.1559" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="288.734,-429.2851 278.918,-425.2981 284.1052,-434.5362 288.734,-429.2851" stroke="#000000"></polygon>
</g>
<!-- 10 -->
<g class="node" id="node11">
<title>10</title>
<polygon fill="#e58139" points="440,-417.5 327,-417.5 327,-349.5 440,-349.5 440,-417.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="383.5" y="-402.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="383.5" y="-387.3">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="383.5" y="-372.3">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="383.5" y="-357.3">class = 악성</text>
</g>
<!-- 2&#45;&gt;10 -->
<g class="edge" id="edge10">
<title>2-&gt;10</title>
<path d="M372.4458,-460.8796C373.9695,-450.2134 375.614,-438.7021 377.1569,-427.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="380.6484,-428.2094 378.5979,-417.8149 373.7187,-427.2194 380.6484,-428.2094" stroke="#000000"></polygon>
</g>
<!-- 4 -->
<g class="node" id="node5">
<title>4</title>
<polygon fill="#3b9ee5" points="248.5,-306 32.5,-306 32.5,-223 248.5,-223 248.5,-306" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="140.5" y="-290.8">smoothness error &lt;= 0.003</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="140.5" y="-275.8">gini = 0.016</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="140.5" y="-260.8">samples = 247</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="140.5" y="-245.8">value = [2, 245]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="140.5" y="-230.8">class = 양성</text>
</g>
<!-- 3&#45;&gt;4 -->
<g class="edge" id="edge4">
<title>3-&gt;4</title>
<path d="M199.6726,-341.8796C192.8543,-332.9633 185.5844,-323.4565 178.5579,-314.268" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="181.318,-312.1156 172.4632,-306.2981 175.7575,-316.3678 181.318,-312.1156" stroke="#000000"></polygon>
</g>
<!-- 9 -->
<g class="node" id="node10">
<title>9</title>
<polygon fill="#7bbeee" points="380,-298.5 267,-298.5 267,-230.5 380,-230.5 380,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="323.5" y="-283.3">gini = 0.375</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="323.5" y="-268.3">samples = 4</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="323.5" y="-253.3">value = [1, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="323.5" y="-238.3">class = 양성</text>
</g>
<!-- 3&#45;&gt;9 -->
<g class="edge" id="edge9">
<title>3-&gt;9</title>
<path d="M263.6771,-341.8796C272.4333,-330.5536 281.9262,-318.2748 290.7156,-306.9058" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="293.6234,-308.8671 296.9708,-298.8149 288.0854,-304.5856 293.6234,-308.8671" stroke="#000000"></polygon>
</g>
<!-- 5 -->
<g class="node" id="node6">
<title>5</title>
<polygon fill="#7bbeee" points="113,-179.5 0,-179.5 0,-111.5 113,-111.5 113,-179.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="56.5" y="-164.3">gini = 0.375</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="56.5" y="-149.3">samples = 4</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="56.5" y="-134.3">value = [1, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="56.5" y="-119.3">class = 양성</text>
</g>
<!-- 4&#45;&gt;5 -->
<g class="edge" id="edge5">
<title>4-&gt;5</title>
<path d="M111.1209,-222.8796C103.2037,-211.6636 94.6269,-199.5131 86.6675,-188.2372" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="89.3486,-185.9662 80.7223,-179.8149 83.6298,-190.003 89.3486,-185.9662" stroke="#000000"></polygon>
</g>
<!-- 6 -->
<g class="node" id="node7">
<title>6</title>
<polygon fill="#3a9de5" points="319.5,-187 131.5,-187 131.5,-104 319.5,-104 319.5,-187" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="225.5" y="-171.8">worst texture &lt;= 33.35</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="225.5" y="-156.8">gini = 0.008</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="225.5" y="-141.8">samples = 243</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="225.5" y="-126.8">value = [1, 242]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="225.5" y="-111.8">class = 양성</text>
</g>
<!-- 4&#45;&gt;6 -->
<g class="edge" id="edge6">
<title>4-&gt;6</title>
<path d="M170.2289,-222.8796C176.5333,-214.0534 183.251,-204.6485 189.7524,-195.5466" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="192.6799,-197.4698 195.6442,-187.2981 186.9837,-193.4011 192.6799,-197.4698" stroke="#000000"></polygon>
</g>
<!-- 7 -->
<g class="node" id="node8">
<title>7</title>
<polygon fill="#399de5" points="219,-68 88,-68 88,0 219,0 219,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="153.5" y="-52.8">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="153.5" y="-37.8">samples = 225</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="153.5" y="-22.8">value = [0, 225]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="153.5" y="-7.8">class = 양성</text>
</g>
<!-- 6&#45;&gt;7 -->
<g class="edge" id="edge7">
<title>6-&gt;7</title>
<path d="M198.6899,-103.9815C192.9331,-95.0666 186.8404,-85.6313 181.0559,-76.6734" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="183.9904,-74.7658 175.6254,-68.2637 178.1099,-78.5631 183.9904,-74.7658" stroke="#000000"></polygon>
</g>
<!-- 8 -->
<g class="node" id="node9">
<title>8</title>
<polygon fill="#45a3e7" points="359.5,-68 237.5,-68 237.5,0 359.5,0 359.5,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="298.5" y="-52.8">gini = 0.105</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="298.5" y="-37.8">samples = 18</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="298.5" y="-22.8">value = [1, 17]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="298.5" y="-7.8">class = 양성</text>
</g>
<!-- 6&#45;&gt;8 -->
<g class="edge" id="edge8">
<title>6-&gt;8</title>
<path d="M252.6825,-103.9815C258.5192,-95.0666 264.6966,-85.6313 270.5614,-76.6734" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="273.5179,-78.5472 276.0673,-68.2637 267.6614,-74.7129 273.5179,-78.5472" stroke="#000000"></polygon>
</g>
<!-- 12 -->
<g class="node" id="node13">
<title>12</title>
<polygon fill="#7bbeee" points="605.5,-417.5 491.5,-417.5 491.5,-349.5 605.5,-349.5 605.5,-417.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="548.5" y="-402.3">gini = 0.375</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="548.5" y="-387.3">samples = 12</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="548.5" y="-372.3">value = [3, 9]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="548.5" y="-357.3">class = 양성</text>
</g>
<!-- 11&#45;&gt;12 -->
<g class="edge" id="edge12">
<title>11-&gt;12</title>
<path d="M558.904,-460.8796C557.4699,-450.2134 555.9221,-438.7021 554.47,-427.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="557.9152,-427.2593 553.1138,-417.8149 550.9776,-428.1922 557.9152,-427.2593" stroke="#000000"></polygon>
</g>
<!-- 13 -->
<g class="node" id="node14">
<title>13</title>
<polygon fill="#e88f4f" points="831,-425 624,-425 624,-342 831,-342 831,-425" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="727.5" y="-409.8">worst symmetry &lt;= 0.268</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="727.5" y="-394.8">gini = 0.18</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="727.5" y="-379.8">samples = 20</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="727.5" y="-364.8">value = [18, 2]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="727.5" y="-349.8">class = 악성</text>
</g>
<!-- 11&#45;&gt;13 -->
<g class="edge" id="edge13">
<title>11-&gt;13</title>
<path d="M621.5095,-460.8796C634.6621,-451.2774 648.7528,-440.9903 662.2268,-431.1534" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="664.5098,-433.8202 670.5227,-425.0969 660.3823,-428.1666 664.5098,-433.8202" stroke="#000000"></polygon>
</g>
<!-- 14 -->
<g class="node" id="node15">
<title>14</title>
<polygon fill="#9ccef2" points="716,-298.5 603,-298.5 603,-230.5 716,-230.5 716,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="659.5" y="-283.3">gini = 0.444</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="659.5" y="-268.3">samples = 3</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="659.5" y="-253.3">value = [1, 2]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="659.5" y="-238.3">class = 양성</text>
</g>
<!-- 13&#45;&gt;14 -->
<g class="edge" id="edge14">
<title>13-&gt;14</title>
<path d="M703.7169,-341.8796C697.3706,-330.7735 690.5007,-318.7513 684.1109,-307.5691" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="687.1088,-305.7609 679.1085,-298.8149 681.0311,-309.2339 687.1088,-305.7609" stroke="#000000"></polygon>
</g>
<!-- 15 -->
<g class="node" id="node16">
<title>15</title>
<polygon fill="#e58139" points="856.5,-298.5 734.5,-298.5 734.5,-230.5 856.5,-230.5 856.5,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="795.5" y="-283.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="795.5" y="-268.3">samples = 17</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="795.5" y="-253.3">value = [17, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="795.5" y="-238.3">class = 악성</text>
</g>
<!-- 13&#45;&gt;15 -->
<g class="edge" id="edge15">
<title>13-&gt;15</title>
<path d="M751.2831,-341.8796C757.6294,-330.7735 764.4993,-318.7513 770.8891,-307.5691" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="773.9689,-309.2339 775.8915,-298.8149 767.8912,-305.7609 773.9689,-309.2339" stroke="#000000"></polygon>
</g>
<!-- 17 -->
<g class="node" id="node18">
<title>17</title>
<polygon fill="#399de5" points="820,-536.5 707,-536.5 707,-468.5 820,-468.5 820,-536.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="763.5" y="-521.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="763.5" y="-506.3">samples = 5</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="763.5" y="-491.3">value = [0, 5]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="763.5" y="-476.3">class = 양성</text>
</g>
<!-- 16&#45;&gt;17 -->
<g class="edge" id="edge17">
<title>16-&gt;17</title>
<path d="M777.1552,-579.8796C775.2729,-569.2134 773.2415,-557.7021 771.3356,-546.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="774.7403,-546.0545 769.5556,-536.8149 767.8468,-547.2711 774.7403,-546.0545" stroke="#000000"></polygon>
</g>
<!-- 18 -->
<g class="node" id="node19">
<title>18</title>
<polygon fill="#e6843d" points="1042.5,-544 838.5,-544 838.5,-461 1042.5,-461 1042.5,-544" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="940.5" y="-528.8">worst concavity &lt;= 0.191</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="940.5" y="-513.8">gini = 0.043</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="940.5" y="-498.8">samples = 137</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="940.5" y="-483.8">value = [134, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="940.5" y="-468.8">class = 악성</text>
</g>
<!-- 16&#45;&gt;18 -->
<g class="edge" id="edge18">
<title>16-&gt;18</title>
<path d="M839.0612,-579.8796C851.5302,-570.368 864.8802,-560.1843 877.6648,-550.432" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="880.1415,-552.9448 885.9696,-544.0969 875.896,-547.3792 880.1415,-552.9448" stroke="#000000"></polygon>
</g>
<!-- 19 -->
<g class="node" id="node20">
<title>19</title>
<polygon fill="#bddef6" points="987,-417.5 874,-417.5 874,-349.5 987,-349.5 987,-417.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="930.5" y="-402.3">gini = 0.48</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="930.5" y="-387.3">samples = 5</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="930.5" y="-372.3">value = [2, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="930.5" y="-357.3">class = 양성</text>
</g>
<!-- 18&#45;&gt;19 -->
<g class="edge" id="edge19">
<title>18-&gt;19</title>
<path d="M937.0025,-460.8796C936.1062,-450.2134 935.1388,-438.7021 934.2312,-427.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="937.7088,-427.4867 933.3836,-417.8149 930.7334,-428.0729 937.7088,-427.4867" stroke="#000000"></polygon>
</g>
<!-- 20 -->
<g class="node" id="node21">
<title>20</title>
<polygon fill="#e58139" points="1136,-417.5 1005,-417.5 1005,-349.5 1136,-349.5 1136,-417.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1070.5" y="-402.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1070.5" y="-387.3">samples = 132</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1070.5" y="-372.3">value = [132, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1070.5" y="-357.3">class = 악성</text>
</g>
<!-- 18&#45;&gt;20 -->
<g class="edge" id="edge20">
<title>18-&gt;20</title>
<path d="M985.9677,-460.8796C998.8211,-449.1138 1012.7979,-436.3197 1025.6166,-424.5855" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1028.0001,-427.1488 1033.0131,-417.8149 1023.2736,-421.9854 1028.0001,-427.1488" stroke="#000000"></polygon>
</g>
</g>
</svg>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h3 id="min_samples_leaf">min_samples_leaf</h3>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><code>min_samples_leaf</code>는 말단 노드의 최소 샘플의 숫자를 지정합니다.</p>
<p>기본 값은 1 입니다.</p>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">DecisionTreeClassifier</span><span class="p">()</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">

<div class="output_text output_subarea output_execute_result">
<pre>DecisionTreeClassifier()</pre>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h3 id="max_leaf_nodes">max_leaf_nodes</h3>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><code>max_leaf_nodes</code>는 말단 노드의 최대 갯수 (과대 적합 방지용)</p>
<p>기본 값은 None, 제한 없음 입니다.</p>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">tree</span> <span class="o">=</span> <span class="n">DecisionTreeClassifier</span><span class="p">(</span><span class="n">max_depth</span><span class="o">=</span><span class="mi">7</span><span class="p">,</span> <span class="n">max_leaf_nodes</span><span class="o">=</span><span class="mi">10</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="n">SEED</span><span class="p">)</span>
<span class="n">tree</span><span class="o">.</span><span class="n">fit</span><span class="p">(</span><span class="n">x_train</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>
<span class="n">show_trees</span><span class="p">(</span><span class="n">tree</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">
<div class="prompt"></div>
<div class="output_subarea output_stream output_stdout output_text">
<pre>정확도: 94.41 %
</pre>
</div>
</div>
<div class="output_area">
<div class="prompt"></div>
<div class="output_svg output_subarea">
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
 "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">

<!-- Generated by graphviz version 2.40.1 (20161225.0304)
 -->
<!-- Title: Tree Pages: 1 -->
<svg height="552pt" viewbox="0.00 0.00 1073.00 552.00" width="1073pt" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<g class="graph" id="graph0" transform="scale(1 1) rotate(0) translate(4 548)">
<title>Tree</title>
<polygon fill="#ffffff" points="-4,4 -4,-548 1069,-548 1069,4 -4,4" stroke="transparent"></polygon>
<!-- 0 -->
<g class="node" id="node1">
<title>0</title>
<polygon fill="#afd7f4" points="622,-544 433,-544 433,-461 622,-461 622,-544" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="527.5" y="-528.8">worst radius &lt;= 16.795</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="527.5" y="-513.8">gini = 0.468</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="527.5" y="-498.8">samples = 426</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="527.5" y="-483.8">value = [159, 267]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="527.5" y="-468.8">class = 양성</text>
</g>
<!-- 1 -->
<g class="node" id="node2">
<title>1</title>
<polygon fill="#4ca6e8" points="533.5,-425 291.5,-425 291.5,-342 533.5,-342 533.5,-425" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="412.5" y="-409.8">worst concave points &lt;= 0.136</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="412.5" y="-394.8">gini = 0.161</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="412.5" y="-379.8">samples = 284</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="412.5" y="-364.8">value = [25, 259]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="412.5" y="-349.8">class = 양성</text>
</g>
<!-- 0&#45;&gt;1 -->
<g class="edge" id="edge1">
<title>0-&gt;1</title>
<path d="M487.2786,-460.8796C478.488,-451.7832 469.1034,-442.0722 460.0574,-432.7116" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="462.3591,-430.0568 452.8931,-425.2981 457.3255,-434.9212 462.3591,-430.0568" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="452.4658" y="-446.5944">True</text>
</g>
<!-- 2 -->
<g class="node" id="node13">
<title>2</title>
<polygon fill="#e78945" points="735.5,-425 551.5,-425 551.5,-342 735.5,-342 735.5,-425" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="643.5" y="-409.8">texture error &lt;= 0.473</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="643.5" y="-394.8">gini = 0.106</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="643.5" y="-379.8">samples = 142</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="643.5" y="-364.8">value = [134, 8]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="643.5" y="-349.8">class = 악성</text>
</g>
<!-- 0&#45;&gt;2 -->
<g class="edge" id="edge12">
<title>0-&gt;2</title>
<path d="M568.0712,-460.8796C576.9382,-451.7832 586.4044,-442.0722 595.5291,-432.7116" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="598.2817,-434.9019 602.7557,-425.2981 593.2691,-430.0158 598.2817,-434.9019" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="603.0748" y="-446.596">False</text>
</g>
<!-- 3 -->
<g class="node" id="node3">
<title>3</title>
<polygon fill="#3c9fe5" points="305,-306 102,-306 102,-223 305,-223 305,-306" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="203.5" y="-290.8">perimeter error &lt;= 6.597</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="203.5" y="-275.8">gini = 0.031</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="203.5" y="-260.8">samples = 252</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="203.5" y="-245.8">value = [4, 248]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="203.5" y="-230.8">class = 양성</text>
</g>
<!-- 1&#45;&gt;3 -->
<g class="edge" id="edge2">
<title>1-&gt;3</title>
<path d="M339.4019,-341.8796C322.0603,-332.0056 303.4467,-321.4075 285.7289,-311.3193" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="286.9787,-308.0034 276.5568,-306.0969 283.5151,-314.0865 286.9787,-308.0034" stroke="#000000"></polygon>
</g>
<!-- 4 -->
<g class="node" id="node6">
<title>4</title>
<polygon fill="#f3c3a1" points="511.5,-306 323.5,-306 323.5,-223 511.5,-223 511.5,-306" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="417.5" y="-290.8">worst texture &lt;= 25.62</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="417.5" y="-275.8">gini = 0.451</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="417.5" y="-260.8">samples = 32</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="417.5" y="-245.8">value = [21, 11]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="417.5" y="-230.8">class = 악성</text>
</g>
<!-- 1&#45;&gt;4 -->
<g class="edge" id="edge5">
<title>1-&gt;4</title>
<path d="M414.2488,-341.8796C414.5931,-333.6838 414.9584,-324.9891 415.3151,-316.5013" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="418.8208,-316.4362 415.7438,-306.2981 411.827,-316.1423 418.8208,-316.4362" stroke="#000000"></polygon>
</g>
<!-- 17 -->
<g class="node" id="node4">
<title>17</title>
<polygon fill="#3b9ee5" points="131,-179.5 0,-179.5 0,-111.5 131,-111.5 131,-179.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="65.5" y="-164.3">gini = 0.024</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="65.5" y="-149.3">samples = 251</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="65.5" y="-134.3">value = [3, 248]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="65.5" y="-119.3">class = 양성</text>
</g>
<!-- 3&#45;&gt;17 -->
<g class="edge" id="edge3">
<title>3-&gt;17</title>
<path d="M155.2343,-222.8796C141.5899,-211.1138 126.7531,-198.3197 113.1454,-186.5855" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="115.1527,-183.6948 105.2938,-179.8149 110.5813,-188.9961 115.1527,-183.6948" stroke="#000000"></polygon>
</g>
<!-- 18 -->
<g class="node" id="node5">
<title>18</title>
<polygon fill="#e58139" points="262,-179.5 149,-179.5 149,-111.5 262,-111.5 262,-179.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="205.5" y="-164.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="205.5" y="-149.3">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="205.5" y="-134.3">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="205.5" y="-119.3">class = 악성</text>
</g>
<!-- 3&#45;&gt;18 -->
<g class="edge" id="edge4">
<title>3-&gt;18</title>
<path d="M204.1995,-222.8796C204.3788,-212.2134 204.5722,-200.7021 204.7538,-189.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="208.2546,-189.8724 204.9233,-179.8149 201.2556,-189.7547 208.2546,-189.8724" stroke="#000000"></polygon>
</g>
<!-- 7 -->
<g class="node" id="node7">
<title>7</title>
<polygon fill="#7bbeee" points="466,-187 299,-187 299,-104 466,-104 466,-187" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="382.5" y="-171.8">worst area &lt;= 817.1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="382.5" y="-156.8">gini = 0.375</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="382.5" y="-141.8">samples = 12</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="382.5" y="-126.8">value = [3, 9]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="382.5" y="-111.8">class = 양성</text>
</g>
<!-- 4&#45;&gt;7 -->
<g class="edge" id="edge6">
<title>4-&gt;7</title>
<path d="M405.2587,-222.8796C402.7952,-214.5037 400.1784,-205.6067 397.63,-196.942" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="400.9731,-195.9041 394.7936,-187.2981 394.2575,-197.8793 400.9731,-195.9041" stroke="#000000"></polygon>
</g>
<!-- 8 -->
<g class="node" id="node10">
<title>8</title>
<polygon fill="#e88f4f" points="691,-187 484,-187 484,-104 691,-104 691,-187" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="587.5" y="-171.8">worst symmetry &lt;= 0.268</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="587.5" y="-156.8">gini = 0.18</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="587.5" y="-141.8">samples = 20</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="587.5" y="-126.8">value = [18, 2]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="587.5" y="-111.8">class = 악성</text>
</g>
<!-- 4&#45;&gt;8 -->
<g class="edge" id="edge9">
<title>4-&gt;8</title>
<path d="M476.9578,-222.8796C490.8046,-213.1868 505.6484,-202.7961 519.8212,-192.8752" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="521.8906,-195.6989 528.0758,-187.0969 517.8764,-189.9643 521.8906,-195.6989" stroke="#000000"></polygon>
</g>
<!-- 11 -->
<g class="node" id="node8">
<title>11</title>
<polygon fill="#4fa8e8" points="315.5,-68 201.5,-68 201.5,0 315.5,0 315.5,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="258.5" y="-52.8">gini = 0.18</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="258.5" y="-37.8">samples = 10</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="258.5" y="-22.8">value = [1, 9]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="258.5" y="-7.8">class = 양성</text>
</g>
<!-- 7&#45;&gt;11 -->
<g class="edge" id="edge7">
<title>7-&gt;11</title>
<path d="M336.327,-103.9815C325.7993,-94.5151 314.6192,-84.462 304.1151,-75.0168" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="306.3811,-72.3475 296.6049,-68.2637 301.7006,-77.5526 306.3811,-72.3475" stroke="#000000"></polygon>
</g>
<!-- 12 -->
<g class="node" id="node9">
<title>12</title>
<polygon fill="#e58139" points="447,-68 334,-68 334,0 447,0 447,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="390.5" y="-52.8">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="390.5" y="-37.8">samples = 2</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="390.5" y="-22.8">value = [2, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="390.5" y="-7.8">class = 악성</text>
</g>
<!-- 7&#45;&gt;12 -->
<g class="edge" id="edge8">
<title>7-&gt;12</title>
<path d="M385.4789,-103.9815C386.079,-95.618 386.7119,-86.7965 387.3186,-78.3409" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="390.8169,-78.4885 388.0416,-68.2637 383.8349,-77.9875 390.8169,-78.4885" stroke="#000000"></polygon>
</g>
<!-- 15 -->
<g class="node" id="node11">
<title>15</title>
<polygon fill="#9ccef2" points="589,-68 476,-68 476,0 589,0 589,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="532.5" y="-52.8">gini = 0.444</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="532.5" y="-37.8">samples = 3</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="532.5" y="-22.8">value = [1, 2]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="532.5" y="-7.8">class = 양성</text>
</g>
<!-- 8&#45;&gt;15 -->
<g class="edge" id="edge10">
<title>8-&gt;15</title>
<path d="M567.02,-103.9815C562.7585,-95.3423 558.256,-86.2144 553.9603,-77.5059" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="556.9641,-75.6836 549.4014,-68.2637 550.6863,-78.7803 556.9641,-75.6836" stroke="#000000"></polygon>
</g>
<!-- 16 -->
<g class="node" id="node12">
<title>16</title>
<polygon fill="#e58139" points="729.5,-68 607.5,-68 607.5,0 729.5,0 729.5,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="668.5" y="-52.8">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="668.5" y="-37.8">samples = 17</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="668.5" y="-22.8">value = [17, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="668.5" y="-7.8">class = 악성</text>
</g>
<!-- 8&#45;&gt;16 -->
<g class="edge" id="edge11">
<title>8-&gt;16</title>
<path d="M617.6614,-103.9815C624.2045,-94.9747 631.1334,-85.4367 637.7008,-76.3965" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="640.5631,-78.4113 643.6089,-68.2637 634.8998,-74.2971 640.5631,-78.4113" stroke="#000000"></polygon>
</g>
<!-- 5 -->
<g class="node" id="node14">
<title>5</title>
<polygon fill="#399de5" points="698,-298.5 585,-298.5 585,-230.5 698,-230.5 698,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="641.5" y="-283.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="641.5" y="-268.3">samples = 5</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="641.5" y="-253.3">value = [0, 5]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="641.5" y="-238.3">class = 양성</text>
</g>
<!-- 2&#45;&gt;5 -->
<g class="edge" id="edge13">
<title>2-&gt;5</title>
<path d="M642.8005,-341.8796C642.6212,-331.2134 642.4278,-319.7021 642.2462,-308.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="645.7444,-308.7547 642.0767,-298.8149 638.7454,-308.8724 645.7444,-308.7547" stroke="#000000"></polygon>
</g>
<!-- 6 -->
<g class="node" id="node15">
<title>6</title>
<polygon fill="#e6843d" points="920.5,-306 716.5,-306 716.5,-223 920.5,-223 920.5,-306" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="818.5" y="-290.8">worst concavity &lt;= 0.191</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="818.5" y="-275.8">gini = 0.043</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="818.5" y="-260.8">samples = 137</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="818.5" y="-245.8">value = [134, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="818.5" y="-230.8">class = 악성</text>
</g>
<!-- 2&#45;&gt;6 -->
<g class="edge" id="edge14">
<title>2-&gt;6</title>
<path d="M704.7065,-341.8796C718.9606,-332.1868 734.241,-321.7961 748.8307,-311.8752" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="751.0269,-314.6143 757.3281,-306.0969 747.0907,-308.8258 751.0269,-314.6143" stroke="#000000"></polygon>
</g>
<!-- 9 -->
<g class="node" id="node16">
<title>9</title>
<polygon fill="#bddef6" points="916,-187 719,-187 719,-104 916,-104 916,-187" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="817.5" y="-171.8">worst texture &lt;= 30.975</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="817.5" y="-156.8">gini = 0.48</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="817.5" y="-141.8">samples = 5</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="817.5" y="-126.8">value = [2, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="817.5" y="-111.8">class = 양성</text>
</g>
<!-- 6&#45;&gt;9 -->
<g class="edge" id="edge15">
<title>6-&gt;9</title>
<path d="M818.1502,-222.8796C818.0814,-214.6838 818.0083,-205.9891 817.937,-197.5013" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="821.4352,-197.2683 817.8512,-187.2981 814.4355,-197.3272 821.4352,-197.2683" stroke="#000000"></polygon>
</g>
<!-- 10 -->
<g class="node" id="node19">
<title>10</title>
<polygon fill="#e58139" points="1065,-179.5 934,-179.5 934,-111.5 1065,-111.5 1065,-179.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="999.5" y="-164.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="999.5" y="-149.3">samples = 132</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="999.5" y="-134.3">value = [132, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="999.5" y="-119.3">class = 악성</text>
</g>
<!-- 6&#45;&gt;10 -->
<g class="edge" id="edge18">
<title>6-&gt;10</title>
<path d="M881.805,-222.8796C900.2903,-210.7263 920.4443,-197.4759 938.7665,-185.4297" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="941.1655,-188.0412 947.5986,-179.623 937.3199,-182.1921 941.1655,-188.0412" stroke="#000000"></polygon>
</g>
<!-- 13 -->
<g class="node" id="node17">
<title>13</title>
<polygon fill="#399de5" points="866,-68 753,-68 753,0 866,0 866,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="809.5" y="-52.8">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="809.5" y="-37.8">samples = 3</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="809.5" y="-22.8">value = [0, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="809.5" y="-7.8">class = 양성</text>
</g>
<!-- 9&#45;&gt;13 -->
<g class="edge" id="edge16">
<title>9-&gt;13</title>
<path d="M814.5211,-103.9815C813.921,-95.618 813.2881,-86.7965 812.6814,-78.3409" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="816.1651,-77.9875 811.9584,-68.2637 809.1831,-78.4885 816.1651,-77.9875" stroke="#000000"></polygon>
</g>
<!-- 14 -->
<g class="node" id="node18">
<title>14</title>
<polygon fill="#e58139" points="997,-68 884,-68 884,0 997,0 997,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="940.5" y="-52.8">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="940.5" y="-37.8">samples = 2</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="940.5" y="-22.8">value = [2, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="940.5" y="-7.8">class = 악성</text>
</g>
<!-- 9&#45;&gt;14 -->
<g class="edge" id="edge17">
<title>9-&gt;14</title>
<path d="M863.3007,-103.9815C873.7434,-94.5151 884.8333,-84.462 895.2528,-75.0168" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="897.6441,-77.5731 902.7024,-68.2637 892.9428,-72.3868 897.6441,-77.5731" stroke="#000000"></polygon>
</g>
</g>
</svg>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h3 id="max_features">max_features</h3>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>최적의 분할을 찾기 위해 고려할 피처의 수</p>
<p>0.8 은 80% 의 feature 만 고려하여 분할 알고리즘 적용</p>
<p>기본 값은 None, 모두 사용입니다.</p>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">tree</span> <span class="o">=</span> <span class="n">DecisionTreeClassifier</span><span class="p">(</span><span class="n">max_depth</span><span class="o">=</span><span class="mi">7</span><span class="p">,</span> <span class="n">max_features</span><span class="o">=</span><span class="mf">0.8</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="n">SEED</span><span class="p">)</span>
<span class="n">tree</span><span class="o">.</span><span class="n">fit</span><span class="p">(</span><span class="n">x_train</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>
<span class="n">show_trees</span><span class="p">(</span><span class="n">tree</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">
<div class="prompt"></div>
<div class="output_subarea output_stream output_stdout output_text">
<pre>정확도: 90.91 %
</pre>
</div>
</div>
<div class="output_area">
<div class="prompt"></div>
<div class="output_svg output_subarea">
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
 "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">

<!-- Generated by graphviz version 2.40.1 (20161225.0304)
 -->
<!-- Title: Tree Pages: 1 -->
<svg height="909pt" viewbox="0.00 0.00 1694.00 909.00" width="1694pt" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<g class="graph" id="graph0" transform="scale(1 1) rotate(0) translate(4 905)">
<title>Tree</title>
<polygon fill="#ffffff" points="-4,4 -4,-905 1690,-905 1690,4 -4,4" stroke="transparent"></polygon>
<!-- 0 -->
<g class="node" id="node1">
<title>0</title>
<polygon fill="#afd7f4" points="1216,-901 1027,-901 1027,-818 1216,-818 1216,-901" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1121.5" y="-885.8">worst radius &lt;= 16.795</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1121.5" y="-870.8">gini = 0.468</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1121.5" y="-855.8">samples = 426</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1121.5" y="-840.8">value = [159, 267]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1121.5" y="-825.8">class = 양성</text>
</g>
<!-- 1 -->
<g class="node" id="node2">
<title>1</title>
<polygon fill="#4ca6e8" points="1049.5,-782 807.5,-782 807.5,-699 1049.5,-699 1049.5,-782" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="928.5" y="-766.8">worst concave points &lt;= 0.136</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="928.5" y="-751.8">gini = 0.161</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="928.5" y="-736.8">samples = 284</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="928.5" y="-721.8">value = [25, 259]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="928.5" y="-706.8">class = 양성</text>
</g>
<!-- 0&#45;&gt;1 -->
<g class="edge" id="edge1">
<title>0-&gt;1</title>
<path d="M1053.998,-817.8796C1038.1308,-808.0962 1021.1106,-797.6019 1004.8844,-787.5971" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1006.3129,-784.3661 995.9639,-782.0969 1002.639,-790.3245 1006.3129,-784.3661" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1001.7334" y="-802.7221">True</text>
</g>
<!-- 28 -->
<g class="node" id="node29">
<title>28</title>
<polygon fill="#e78945" points="1271.5,-782 1087.5,-782 1087.5,-699 1271.5,-699 1271.5,-782" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1179.5" y="-766.8">texture error &lt;= 0.473</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1179.5" y="-751.8">gini = 0.106</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1179.5" y="-736.8">samples = 142</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1179.5" y="-721.8">value = [134, 8]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1179.5" y="-706.8">class = 악성</text>
</g>
<!-- 0&#45;&gt;28 -->
<g class="edge" id="edge28">
<title>0-&gt;28</title>
<path d="M1141.7856,-817.8796C1145.9557,-809.3236 1150.3909,-800.2238 1154.6997,-791.3833" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1157.8927,-792.8207 1159.1278,-782.2981 1151.6003,-789.7538 1157.8927,-792.8207" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1167.2735" y="-802.2338">False</text>
</g>
<!-- 2 -->
<g class="node" id="node3">
<title>2</title>
<polygon fill="#3c9fe5" points="581.5,-663 321.5,-663 321.5,-580 581.5,-580 581.5,-663" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-647.8">worst fractal dimension &lt;= 0.055</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-632.8">gini = 0.031</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-617.8">samples = 252</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-602.8">value = [4, 248]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-587.8">class = 양성</text>
</g>
<!-- 1&#45;&gt;2 -->
<g class="edge" id="edge2">
<title>1-&gt;2</title>
<path d="M807.2075,-710.2404C741.5697,-693.8654 660.1694,-673.558 591.8085,-656.5036" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="592.3703,-653.0365 581.8205,-654.0118 590.6759,-659.8284 592.3703,-653.0365" stroke="#000000"></polygon>
</g>
<!-- 17 -->
<g class="node" id="node18">
<title>17</title>
<polygon fill="#f3c3a1" points="1022.5,-663 834.5,-663 834.5,-580 1022.5,-580 1022.5,-663" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="928.5" y="-647.8">worst texture &lt;= 25.62</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="928.5" y="-632.8">gini = 0.451</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="928.5" y="-617.8">samples = 32</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="928.5" y="-602.8">value = [21, 11]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="928.5" y="-587.8">class = 악성</text>
</g>
<!-- 1&#45;&gt;17 -->
<g class="edge" id="edge17">
<title>1-&gt;17</title>
<path d="M928.5,-698.8796C928.5,-690.6838 928.5,-681.9891 928.5,-673.5013" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="932.0001,-673.298 928.5,-663.2981 925.0001,-673.2981 932.0001,-673.298" stroke="#000000"></polygon>
</g>
<!-- 3 -->
<g class="node" id="node4">
<title>3</title>
<polygon fill="#e58139" points="325,-536.5 212,-536.5 212,-468.5 325,-468.5 325,-536.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="268.5" y="-521.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="268.5" y="-506.3">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="268.5" y="-491.3">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="268.5" y="-476.3">class = 악성</text>
</g>
<!-- 2&#45;&gt;3 -->
<g class="edge" id="edge3">
<title>2-&gt;3</title>
<path d="M387.4955,-579.8796C368.6361,-567.6158 348.0586,-554.2348 329.3998,-542.1015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="331.2664,-539.1404 320.9749,-536.623 327.4503,-545.0088 331.2664,-539.1404" stroke="#000000"></polygon>
</g>
<!-- 4 -->
<g class="node" id="node5">
<title>4</title>
<polygon fill="#3b9ee5" points="559.5,-544 343.5,-544 343.5,-461 559.5,-461 559.5,-544" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-528.8">smoothness error &lt;= 0.003</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-513.8">gini = 0.024</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-498.8">samples = 251</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-483.8">value = [3, 248]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-468.8">class = 양성</text>
</g>
<!-- 2&#45;&gt;4 -->
<g class="edge" id="edge4">
<title>2-&gt;4</title>
<path d="M451.5,-579.8796C451.5,-571.6838 451.5,-562.9891 451.5,-554.5013" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="455.0001,-554.298 451.5,-544.2981 448.0001,-554.2981 455.0001,-554.298" stroke="#000000"></polygon>
</g>
<!-- 5 -->
<g class="node" id="node6">
<title>5</title>
<polygon fill="#7bbeee" points="296.5,-425 78.5,-425 78.5,-342 296.5,-342 296.5,-425" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-409.8">mean smoothness &lt;= 0.081</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-394.8">gini = 0.375</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-379.8">samples = 4</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-364.8">value = [1, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-349.8">class = 양성</text>
</g>
<!-- 4&#45;&gt;5 -->
<g class="edge" id="edge5">
<title>4-&gt;5</title>
<path d="M359.1656,-460.8796C336.4565,-450.6433 312.0207,-439.6286 288.9089,-429.2108" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="290.3372,-426.0155 279.7823,-425.0969 287.4606,-432.3972 290.3372,-426.0155" stroke="#000000"></polygon>
</g>
<!-- 8 -->
<g class="node" id="node9">
<title>8</title>
<polygon fill="#3b9ee5" points="528.5,-425 374.5,-425 374.5,-342 528.5,-342 528.5,-425" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-409.8">area error &lt;= 48.7</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-394.8">gini = 0.016</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-379.8">samples = 247</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-364.8">value = [2, 245]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="451.5" y="-349.8">class = 양성</text>
</g>
<!-- 4&#45;&gt;8 -->
<g class="edge" id="edge8">
<title>4-&gt;8</title>
<path d="M451.5,-460.8796C451.5,-452.6838 451.5,-443.9891 451.5,-435.5013" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="455.0001,-435.298 451.5,-425.2981 448.0001,-435.2981 455.0001,-435.298" stroke="#000000"></polygon>
</g>
<!-- 6 -->
<g class="node" id="node7">
<title>6</title>
<polygon fill="#399de5" points="113,-298.5 0,-298.5 0,-230.5 113,-230.5 113,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="56.5" y="-283.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="56.5" y="-268.3">samples = 3</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="56.5" y="-253.3">value = [0, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="56.5" y="-238.3">class = 양성</text>
</g>
<!-- 5&#45;&gt;6 -->
<g class="edge" id="edge6">
<title>5-&gt;6</title>
<path d="M141.6826,-341.8796C128.7303,-330.1138 114.646,-317.3197 101.7286,-305.5855" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="104.0306,-302.9482 94.2753,-298.8149 99.3239,-308.1296 104.0306,-302.9482" stroke="#000000"></polygon>
</g>
<!-- 7 -->
<g class="node" id="node8">
<title>7</title>
<polygon fill="#e58139" points="244,-298.5 131,-298.5 131,-230.5 244,-230.5 244,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-283.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-268.3">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-253.3">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="187.5" y="-238.3">class = 악성</text>
</g>
<!-- 5&#45;&gt;7 -->
<g class="edge" id="edge7">
<title>5-&gt;7</title>
<path d="M187.5,-341.8796C187.5,-331.2134 187.5,-319.7021 187.5,-308.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="191.0001,-308.8149 187.5,-298.8149 184.0001,-308.815 191.0001,-308.8149" stroke="#000000"></polygon>
</g>
<!-- 9 -->
<g class="node" id="node10">
<title>9</title>
<polygon fill="#3a9de5" points="450.5,-306 262.5,-306 262.5,-223 450.5,-223 450.5,-306" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="356.5" y="-290.8">worst texture &lt;= 33.35</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="356.5" y="-275.8">gini = 0.008</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="356.5" y="-260.8">samples = 243</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="356.5" y="-245.8">value = [1, 242]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="356.5" y="-230.8">class = 양성</text>
</g>
<!-- 8&#45;&gt;9 -->
<g class="edge" id="edge9">
<title>8-&gt;9</title>
<path d="M418.2736,-341.8796C411.1556,-332.9633 403.5661,-323.4565 396.2308,-314.268" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="398.8425,-311.9295 389.8682,-306.2981 393.3719,-316.2968 398.8425,-311.9295" stroke="#000000"></polygon>
</g>
<!-- 14 -->
<g class="node" id="node15">
<title>14</title>
<polygon fill="#7bbeee" points="652.5,-306 468.5,-306 468.5,-223 652.5,-223 652.5,-306" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="560.5" y="-290.8">texture error &lt;= 1.938</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="560.5" y="-275.8">gini = 0.375</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="560.5" y="-260.8">samples = 4</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="560.5" y="-245.8">value = [1, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="560.5" y="-230.8">class = 양성</text>
</g>
<!-- 8&#45;&gt;14 -->
<g class="edge" id="edge14">
<title>8-&gt;14</title>
<path d="M489.6229,-341.8796C497.9549,-332.7832 506.8498,-323.0722 515.4239,-313.7116" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="518.0408,-316.0363 522.2144,-306.2981 512.8789,-311.3081 518.0408,-316.0363" stroke="#000000"></polygon>
</g>
<!-- 10 -->
<g class="node" id="node11">
<title>10</title>
<polygon fill="#399de5" points="259,-179.5 128,-179.5 128,-111.5 259,-111.5 259,-179.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="193.5" y="-164.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="193.5" y="-149.3">samples = 225</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="193.5" y="-134.3">value = [0, 225]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="193.5" y="-119.3">class = 양성</text>
</g>
<!-- 9&#45;&gt;10 -->
<g class="edge" id="edge10">
<title>9-&gt;10</title>
<path d="M299.4905,-222.8796C282.9949,-210.8368 265.0237,-197.7167 248.6441,-185.7586" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="250.3804,-182.6927 240.24,-179.623 246.2528,-188.3463 250.3804,-182.6927" stroke="#000000"></polygon>
</g>
<!-- 11 -->
<g class="node" id="node12">
<title>11</title>
<polygon fill="#45a3e7" points="456,-187 277,-187 277,-104 456,-104 456,-187" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="366.5" y="-171.8">worst texture &lt;= 33.8</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="366.5" y="-156.8">gini = 0.105</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="366.5" y="-141.8">samples = 18</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="366.5" y="-126.8">value = [1, 17]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="366.5" y="-111.8">class = 양성</text>
</g>
<!-- 9&#45;&gt;11 -->
<g class="edge" id="edge11">
<title>9-&gt;11</title>
<path d="M359.9975,-222.8796C360.6862,-214.6838 361.4169,-205.9891 362.1301,-197.5013" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="365.6378,-197.5561 362.9876,-187.2981 358.6624,-196.9698 365.6378,-197.5561" stroke="#000000"></polygon>
</g>
<!-- 12 -->
<g class="node" id="node13">
<title>12</title>
<polygon fill="#e58139" points="355,-68 242,-68 242,0 355,0 355,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="298.5" y="-52.8">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="298.5" y="-37.8">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="298.5" y="-22.8">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="298.5" y="-7.8">class = 악성</text>
</g>
<!-- 11&#45;&gt;12 -->
<g class="edge" id="edge12">
<title>11-&gt;12</title>
<path d="M341.1793,-103.9815C335.7984,-95.1585 330.1068,-85.8258 324.6941,-76.9506" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="327.5912,-74.9789 319.3962,-68.2637 321.6149,-78.6236 327.5912,-74.9789" stroke="#000000"></polygon>
</g>
<!-- 13 -->
<g class="node" id="node14">
<title>13</title>
<polygon fill="#399de5" points="495.5,-68 373.5,-68 373.5,0 495.5,0 495.5,-68" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="434.5" y="-52.8">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="434.5" y="-37.8">samples = 17</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="434.5" y="-22.8">value = [0, 17]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="434.5" y="-7.8">class = 양성</text>
</g>
<!-- 11&#45;&gt;13 -->
<g class="edge" id="edge13">
<title>11-&gt;13</title>
<path d="M391.8207,-103.9815C397.2016,-95.1585 402.8932,-85.8258 408.3059,-76.9506" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="411.3851,-78.6236 413.6038,-68.2637 405.4088,-74.9789 411.3851,-78.6236" stroke="#000000"></polygon>
</g>
<!-- 15 -->
<g class="node" id="node16">
<title>15</title>
<polygon fill="#399de5" points="607,-179.5 494,-179.5 494,-111.5 607,-111.5 607,-179.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="550.5" y="-164.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="550.5" y="-149.3">samples = 3</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="550.5" y="-134.3">value = [0, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="550.5" y="-119.3">class = 양성</text>
</g>
<!-- 14&#45;&gt;15 -->
<g class="edge" id="edge15">
<title>14-&gt;15</title>
<path d="M557.0025,-222.8796C556.1062,-212.2134 555.1388,-200.7021 554.2312,-189.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="557.7088,-189.4867 553.3836,-179.8149 550.7334,-190.0729 557.7088,-189.4867" stroke="#000000"></polygon>
</g>
<!-- 16 -->
<g class="node" id="node17">
<title>16</title>
<polygon fill="#e58139" points="738,-179.5 625,-179.5 625,-111.5 738,-111.5 738,-179.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="681.5" y="-164.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="681.5" y="-149.3">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="681.5" y="-134.3">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="681.5" y="-119.3">class = 악성</text>
</g>
<!-- 14&#45;&gt;16 -->
<g class="edge" id="edge16">
<title>14-&gt;16</title>
<path d="M602.8199,-222.8796C614.6717,-211.2237 627.5495,-198.5587 639.3891,-186.9148" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="641.9327,-189.3223 646.6083,-179.8149 637.0244,-184.3314 641.9327,-189.3223" stroke="#000000"></polygon>
</g>
<!-- 18 -->
<g class="node" id="node19">
<title>18</title>
<polygon fill="#7bbeee" points="1005.5,-544 773.5,-544 773.5,-461 1005.5,-461 1005.5,-544" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="889.5" y="-528.8">mean concave points &lt;= 0.08</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="889.5" y="-513.8">gini = 0.375</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="889.5" y="-498.8">samples = 12</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="889.5" y="-483.8">value = [3, 9]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="889.5" y="-468.8">class = 양성</text>
</g>
<!-- 17&#45;&gt;18 -->
<g class="edge" id="edge18">
<title>17-&gt;18</title>
<path d="M914.8597,-579.8796C912.1147,-571.5037 909.1988,-562.6067 906.3591,-553.942" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="909.6389,-552.7107 903.1985,-544.2981 902.987,-554.8908 909.6389,-552.7107" stroke="#000000"></polygon>
</g>
<!-- 23 -->
<g class="node" id="node24">
<title>23</title>
<polygon fill="#e88f4f" points="1231,-544 1024,-544 1024,-461 1231,-461 1231,-544" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1127.5" y="-528.8">worst symmetry &lt;= 0.268</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1127.5" y="-513.8">gini = 0.18</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1127.5" y="-498.8">samples = 20</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1127.5" y="-483.8">value = [18, 2]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1127.5" y="-468.8">class = 악성</text>
</g>
<!-- 17&#45;&gt;23 -->
<g class="edge" id="edge23">
<title>17-&gt;23</title>
<path d="M998.1005,-579.8796C1014.6125,-570.0056 1032.3354,-559.4075 1049.2055,-549.3193" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1051.1525,-552.2331 1057.9388,-544.0969 1047.5599,-546.2253 1051.1525,-552.2331" stroke="#000000"></polygon>
</g>
<!-- 19 -->
<g class="node" id="node20">
<title>19</title>
<polygon fill="#4fa8e8" points="848.5,-425 606.5,-425 606.5,-342 848.5,-342 848.5,-425" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="727.5" y="-409.8">worst concave points &lt;= 0.138</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="727.5" y="-394.8">gini = 0.18</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="727.5" y="-379.8">samples = 10</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="727.5" y="-364.8">value = [1, 9]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="727.5" y="-349.8">class = 양성</text>
</g>
<!-- 18&#45;&gt;19 -->
<g class="edge" id="edge19">
<title>18-&gt;19</title>
<path d="M832.8403,-460.8796C819.7684,-451.2774 805.7641,-440.9903 792.3727,-431.1534" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="794.2591,-428.1963 784.1277,-425.0969 790.115,-433.8378 794.2591,-428.1963" stroke="#000000"></polygon>
</g>
<!-- 22 -->
<g class="node" id="node23">
<title>22</title>
<polygon fill="#e58139" points="980,-417.5 867,-417.5 867,-349.5 980,-349.5 980,-417.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="923.5" y="-402.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="923.5" y="-387.3">samples = 2</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="923.5" y="-372.3">value = [2, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="923.5" y="-357.3">class = 악성</text>
</g>
<!-- 18&#45;&gt;22 -->
<g class="edge" id="edge22">
<title>18-&gt;22</title>
<path d="M901.3916,-460.8796C904.4705,-450.1034 907.7958,-438.4647 910.9092,-427.5677" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="914.3138,-428.3917 913.6957,-417.8149 907.5831,-426.4686 914.3138,-428.3917" stroke="#000000"></polygon>
</g>
<!-- 20 -->
<g class="node" id="node21">
<title>20</title>
<polygon fill="#e58139" points="784,-298.5 671,-298.5 671,-230.5 784,-230.5 784,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="727.5" y="-283.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="727.5" y="-268.3">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="727.5" y="-253.3">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="727.5" y="-238.3">class = 악성</text>
</g>
<!-- 19&#45;&gt;20 -->
<g class="edge" id="edge20">
<title>19-&gt;20</title>
<path d="M727.5,-341.8796C727.5,-331.2134 727.5,-319.7021 727.5,-308.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="731.0001,-308.8149 727.5,-298.8149 724.0001,-308.815 731.0001,-308.8149" stroke="#000000"></polygon>
</g>
<!-- 21 -->
<g class="node" id="node22">
<title>21</title>
<polygon fill="#399de5" points="915,-298.5 802,-298.5 802,-230.5 915,-230.5 915,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="858.5" y="-283.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="858.5" y="-268.3">samples = 9</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="858.5" y="-253.3">value = [0, 9]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="858.5" y="-238.3">class = 양성</text>
</g>
<!-- 19&#45;&gt;21 -->
<g class="edge" id="edge21">
<title>19-&gt;21</title>
<path d="M773.3174,-341.8796C786.2697,-330.1138 800.354,-317.3197 813.2714,-305.5855" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="815.6761,-308.1296 820.7247,-298.8149 810.9694,-302.9482 815.6761,-308.1296" stroke="#000000"></polygon>
</g>
<!-- 24 -->
<g class="node" id="node25">
<title>24</title>
<polygon fill="#9ccef2" points="1218.5,-425 998.5,-425 998.5,-342 1218.5,-342 1218.5,-425" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1108.5" y="-409.8">worst smoothness &lt;= 0.132</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1108.5" y="-394.8">gini = 0.444</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1108.5" y="-379.8">samples = 3</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1108.5" y="-364.8">value = [1, 2]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1108.5" y="-349.8">class = 양성</text>
</g>
<!-- 23&#45;&gt;24 -->
<g class="edge" id="edge24">
<title>23-&gt;24</title>
<path d="M1120.8547,-460.8796C1119.5318,-452.5938 1118.1274,-443.798 1116.7581,-435.2216" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1120.2066,-434.6211 1115.1736,-425.2981 1113.2942,-435.7249 1120.2066,-434.6211" stroke="#000000"></polygon>
</g>
<!-- 27 -->
<g class="node" id="node28">
<title>27</title>
<polygon fill="#e58139" points="1358.5,-417.5 1236.5,-417.5 1236.5,-349.5 1358.5,-349.5 1358.5,-417.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1297.5" y="-402.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1297.5" y="-387.3">samples = 17</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1297.5" y="-372.3">value = [17, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1297.5" y="-357.3">class = 악성</text>
</g>
<!-- 23&#45;&gt;27 -->
<g class="edge" id="edge27">
<title>23-&gt;27</title>
<path d="M1186.9578,-460.8796C1204.3196,-448.7263 1223.2488,-435.4759 1240.4575,-423.4297" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1242.5676,-426.225 1248.7528,-417.623 1238.5533,-420.4904 1242.5676,-426.225" stroke="#000000"></polygon>
</g>
<!-- 25 -->
<g class="node" id="node26">
<title>25</title>
<polygon fill="#e58139" points="1106,-298.5 993,-298.5 993,-230.5 1106,-230.5 1106,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1049.5" y="-283.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1049.5" y="-268.3">samples = 1</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1049.5" y="-253.3">value = [1, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1049.5" y="-238.3">class = 악성</text>
</g>
<!-- 24&#45;&gt;25 -->
<g class="edge" id="edge25">
<title>24-&gt;25</title>
<path d="M1087.8647,-341.8796C1082.4128,-330.8835 1076.5157,-318.9893 1071.0184,-307.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1074.0911,-306.2195 1066.5133,-298.8149 1067.8196,-309.3289 1074.0911,-306.2195" stroke="#000000"></polygon>
</g>
<!-- 26 -->
<g class="node" id="node27">
<title>26</title>
<polygon fill="#399de5" points="1237,-298.5 1124,-298.5 1124,-230.5 1237,-230.5 1237,-298.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1180.5" y="-283.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1180.5" y="-268.3">samples = 2</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1180.5" y="-253.3">value = [0, 2]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1180.5" y="-238.3">class = 양성</text>
</g>
<!-- 24&#45;&gt;26 -->
<g class="edge" id="edge26">
<title>24-&gt;26</title>
<path d="M1133.6821,-341.8796C1140.4017,-330.7735 1147.6757,-318.7513 1154.4414,-307.5691" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1157.5559,-309.1826 1159.738,-298.8149 1151.5668,-305.5589 1157.5559,-309.1826" stroke="#000000"></polygon>
</g>
<!-- 29 -->
<g class="node" id="node30">
<title>29</title>
<polygon fill="#399de5" points="1236,-655.5 1123,-655.5 1123,-587.5 1236,-587.5 1236,-655.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1179.5" y="-640.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1179.5" y="-625.3">samples = 5</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1179.5" y="-610.3">value = [0, 5]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1179.5" y="-595.3">class = 양성</text>
</g>
<!-- 28&#45;&gt;29 -->
<g class="edge" id="edge29">
<title>28-&gt;29</title>
<path d="M1179.5,-698.8796C1179.5,-688.2134 1179.5,-676.7021 1179.5,-665.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1183.0001,-665.8149 1179.5,-655.8149 1176.0001,-665.815 1183.0001,-665.8149" stroke="#000000"></polygon>
</g>
<!-- 30 -->
<g class="node" id="node31">
<title>30</title>
<polygon fill="#e6843d" points="1535.5,-663 1331.5,-663 1331.5,-580 1535.5,-580 1535.5,-663" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1433.5" y="-647.8">worst concavity &lt;= 0.191</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1433.5" y="-632.8">gini = 0.043</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1433.5" y="-617.8">samples = 137</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1433.5" y="-602.8">value = [134, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1433.5" y="-587.8">class = 악성</text>
</g>
<!-- 28&#45;&gt;30 -->
<g class="edge" id="edge30">
<title>28-&gt;30</title>
<path d="M1268.3369,-698.8796C1290.0891,-688.6886 1313.488,-677.7261 1335.6371,-667.3492" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1337.1427,-670.5089 1344.7133,-663.0969 1334.1729,-664.1701 1337.1427,-670.5089" stroke="#000000"></polygon>
</g>
<!-- 31 -->
<g class="node" id="node32">
<title>31</title>
<polygon fill="#bddef6" points="1537,-544 1330,-544 1330,-461 1537,-461 1537,-544" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1433.5" y="-528.8">worst symmetry &lt;= 0.284</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1433.5" y="-513.8">gini = 0.48</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1433.5" y="-498.8">samples = 5</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1433.5" y="-483.8">value = [2, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1433.5" y="-468.8">class = 양성</text>
</g>
<!-- 30&#45;&gt;31 -->
<g class="edge" id="edge31">
<title>30-&gt;31</title>
<path d="M1433.5,-579.8796C1433.5,-571.6838 1433.5,-562.9891 1433.5,-554.5013" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1437.0001,-554.298 1433.5,-544.2981 1430.0001,-554.2981 1437.0001,-554.298" stroke="#000000"></polygon>
</g>
<!-- 34 -->
<g class="node" id="node35">
<title>34</title>
<polygon fill="#e58139" points="1686,-536.5 1555,-536.5 1555,-468.5 1686,-468.5 1686,-536.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1620.5" y="-521.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1620.5" y="-506.3">samples = 132</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1620.5" y="-491.3">value = [132, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1620.5" y="-476.3">class = 악성</text>
</g>
<!-- 30&#45;&gt;34 -->
<g class="edge" id="edge34">
<title>30-&gt;34</title>
<path d="M1498.9035,-579.8796C1518.1752,-567.6158 1539.2024,-554.2348 1558.269,-542.1015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1560.3205,-544.9447 1566.8781,-536.623 1556.5624,-539.039 1560.3205,-544.9447" stroke="#000000"></polygon>
</g>
<!-- 32 -->
<g class="node" id="node33">
<title>32</title>
<polygon fill="#399de5" points="1490,-417.5 1377,-417.5 1377,-349.5 1490,-349.5 1490,-417.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1433.5" y="-402.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1433.5" y="-387.3">samples = 3</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1433.5" y="-372.3">value = [0, 3]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1433.5" y="-357.3">class = 양성</text>
</g>
<!-- 31&#45;&gt;32 -->
<g class="edge" id="edge32">
<title>31-&gt;32</title>
<path d="M1433.5,-460.8796C1433.5,-450.2134 1433.5,-438.7021 1433.5,-427.9015" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1437.0001,-427.8149 1433.5,-417.8149 1430.0001,-427.815 1437.0001,-427.8149" stroke="#000000"></polygon>
</g>
<!-- 33 -->
<g class="node" id="node34">
<title>33</title>
<polygon fill="#e58139" points="1621,-417.5 1508,-417.5 1508,-349.5 1621,-349.5 1621,-417.5" stroke="#000000"></polygon>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1564.5" y="-402.3">gini = 0.0</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1564.5" y="-387.3">samples = 2</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1564.5" y="-372.3">value = [2, 0]</text>
<text fill="#000000" font-family="Times,serif" font-size="14.00" text-anchor="middle" x="1564.5" y="-357.3">class = 악성</text>
</g>
<!-- 31&#45;&gt;33 -->
<g class="edge" id="edge33">
<title>31-&gt;33</title>
<path d="M1479.3174,-460.8796C1492.2697,-449.1138 1506.354,-436.3197 1519.2714,-424.5855" fill="none" stroke="#000000"></path>
<polygon fill="#000000" points="1521.6761,-427.1296 1526.7247,-417.8149 1516.9694,-421.9482 1521.6761,-427.1296" stroke="#000000"></polygon>
</g>
</g>
</svg>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="feature의-중요도-파악">feature의 중요도 파악</h2>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><code>feature_importances_</code> 변수를 통해서 tree 알고리즘이 학습시 고려한 feature 별 중요도를 확인할 수 있습니다.</p>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">tree</span><span class="o">.</span><span class="n">feature_importances_</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">

<div class="output_text output_subarea output_execute_result">
<pre>array([0.        , 0.        , 0.        , 0.        , 0.00752597,
       0.        , 0.        , 0.01354675, 0.        , 0.        ,
       0.        , 0.05383566, 0.        , 0.00238745, 0.00231135,
       0.        , 0.        , 0.        , 0.        , 0.        ,
       0.69546322, 0.04179055, 0.        , 0.        , 0.00668975,
       0.        , 0.01740312, 0.12587473, 0.02341413, 0.00975731])</pre>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>DataFrame으로 만들면 <strong>중요도(feature importances) 순서로 정렬</strong>할 수 있습니다.</p>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">df</span> <span class="o">=</span> <span class="n">pd</span><span class="o">.</span><span class="n">DataFrame</span><span class="p">(</span><span class="nb">list</span><span class="p">(</span><span class="nb">zip</span><span class="p">(</span><span class="n">cancer</span><span class="p">[</span><span class="s1">'feature_names'</span><span class="p">],</span> <span class="n">tree</span><span class="o">.</span><span class="n">feature_importances_</span><span class="p">)),</span> <span class="n">columns</span><span class="o">=</span><span class="p">[</span><span class="s1">'feature'</span><span class="p">,</span> <span class="s1">'importance'</span><span class="p">])</span><span class="o">.</span><span class="n">sort_values</span><span class="p">(</span><span class="s1">'importance'</span><span class="p">,</span> <span class="n">ascending</span><span class="o">=</span><span class="kc">False</span><span class="p">)</span>
<span class="n">df</span> <span class="o">=</span> <span class="n">df</span><span class="o">.</span><span class="n">reset_index</span><span class="p">(</span><span class="n">drop</span><span class="o">=</span><span class="kc">True</span><span class="p">)</span>
<span class="n">df</span><span class="o">.</span><span class="n">head</span><span class="p">(</span><span class="mi">15</span><span class="p">)</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">

<div class="output_html rendered_html output_subarea output_execute_result">
<div>
<style scoped="">
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
<thead>
<tr style="text-align: right;">
<th></th>
<th>feature</th>
<th>importance</th>
</tr>
</thead>
<tbody>
<tr>
<th>0</th>
<td>worst radius</td>
<td>0.695463</td>
</tr>
<tr>
<th>1</th>
<td>worst concave points</td>
<td>0.125875</td>
</tr>
<tr>
<th>2</th>
<td>texture error</td>
<td>0.053836</td>
</tr>
<tr>
<th>3</th>
<td>worst texture</td>
<td>0.041791</td>
</tr>
<tr>
<th>4</th>
<td>worst symmetry</td>
<td>0.023414</td>
</tr>
<tr>
<th>5</th>
<td>worst concavity</td>
<td>0.017403</td>
</tr>
<tr>
<th>6</th>
<td>mean concave points</td>
<td>0.013547</td>
</tr>
<tr>
<th>7</th>
<td>worst fractal dimension</td>
<td>0.009757</td>
</tr>
<tr>
<th>8</th>
<td>mean smoothness</td>
<td>0.007526</td>
</tr>
<tr>
<th>9</th>
<td>worst smoothness</td>
<td>0.006690</td>
</tr>
<tr>
<th>10</th>
<td>area error</td>
<td>0.002387</td>
</tr>
<tr>
<th>11</th>
<td>smoothness error</td>
<td>0.002311</td>
</tr>
<tr>
<th>12</th>
<td>worst perimeter</td>
<td>0.000000</td>
</tr>
<tr>
<th>13</th>
<td>worst area</td>
<td>0.000000</td>
</tr>
<tr>
<th>14</th>
<td>symmetry error</td>
<td>0.000000</td>
</tr>
</tbody>
</table>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
<div class="input_area">
<div class="highlight hl-ipython3"><pre><span></span><span class="n">plt</span><span class="o">.</span><span class="n">figure</span><span class="p">(</span><span class="n">figsize</span><span class="o">=</span><span class="p">(</span><span class="mi">10</span><span class="p">,</span> <span class="mi">10</span><span class="p">))</span>
<span class="n">sns</span><span class="o">.</span><span class="n">barplot</span><span class="p">(</span><span class="n">y</span><span class="o">=</span><span class="s1">'feature'</span><span class="p">,</span> <span class="n">x</span><span class="o">=</span><span class="s1">'importance'</span><span class="p">,</span> <span class="n">data</span><span class="o">=</span><span class="n">df</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">show</span><span class="p">()</span>
</pre></div>
</div>
</div>
</div>
<div class="output_wrapper">
<div class="output">
<div class="output_area">
<div class="prompt"></div>
<div class="output_png output_subarea">
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAssAAAJQCAYAAAB4qSlbAAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAALEgAACxIB0t1+/AAAADl0RVh0U29mdHdhcmUAbWF0cGxvdGxpYiB2ZXJzaW9uIDIuMi4zLCBodHRwOi8vbWF0cGxvdGxpYi5vcmcvIxREBQAAIABJREFUeJzs3XuYnWV59v/vSYhskpCIbAr8wGjYCUgCmQSD7KFoqy8ExCIiCrakUDVCCxRfXhFRK2loqVgqhqiRH7FYQIQikihkMxII2ZAdAaVIWq28bBQIARKyOd8/nnvKYlhrZpJMmN35OY455l73c2+u55n8cc2da82SbSIiIiIi4s226uoAIiIiIiK6qyTLERERERENJFmOiIiIiGggyXJERERERANJliMiIiIiGkiyHBERERHRQJLliIiIiIgGkixHRERERDSQZDkiIiIiooGtuzqA6D122mknDx06tKvDiIiIiGjXggULnrO9c3vjkixHpxk6dCjz58/v6jAiIiIi2iXpPzsyLslydJp1z/6BZ791U1eHERERET3Uzud/oqtDeJPULEdERERENJBkOSIiIiKigSTLERERERENJFnupiSNlXRAJ685U1JTad8taUhnrh8RERHR2yRZ7mKS+jW4NBZoN1mWtElv0rT9p7Zf2JS5EREREX1FkuVNJOkSSeNL+xpJ95X28ZJuKu0zJC2VtEzShJq5qyRdKWkuMEbSVZKWS1oi6WpJhwMnARMlLZI0rNXeUyT9o6QZwARJoyXNkfRw+b5fGbedpJvLuj8EtqtZY4WknSQNlbSspv8iSVeU9viauG7eMk8yIiIiovvKn47bdLOBvwGuBZqAbST1B44AmiXtDkwARgLPA9MljbX9Y2AAsMz25ZJ2BL4D7G/bkobYfkHSncBdtm9tsP++wAm210vaATjK9jpJJwB/B3wEOB94xfbBkg4GFm7kPV4KvMv2mkYlG5LGAeMA/r8d37GRy0dERER0bzlZ3nQLgJGSBgFrgAeokuYjgWZgFDDT9rO21wFTgaPK3PXAbaW9ElgNTJZ0KvBKB/e/xfb60h4M3FJOiK8BDiz9RwE3AdheAizZyHtcAkyV9AlgXb0BtifZbrLd9I6BO2zk8hERERHdW5LlTWR7LbACOAeYQ5UgHwsMAx4F1Mb01S2JbkmkR1Mlz2OBezoYwss17a8AM2wfBPwvYNvaUNtZZx1v/HdQO/dDwHVUp+MLNrU+OiIiIqKnSrK8eWYDF5XvzcB5wCLbBuYCR5e64H7AGcCs1gtIGggMtn03cAEwolx6CRjUwTgGA/9d2me3iu/Mss9BwMF15j4N7CLpHZK2AT5cxm8F7Gl7BnAJMAQY2MF4IiIiInqFJMubpxnYDXjA9tNU5RTNALafAr4AzAAWAwtt31FnjUHAXZKWUCXTF5b+m4GLy5v2htWZV+vvga9Luh+o/esa3wIGlrUvAR5qPbGckF9JldzfBTxWLvUDbpK0FHgYuCZ/PSMiIiL6GlWHoBGbb8Q73+2fXXplV4cRERERPdTO53/iLdtL0gLbTe2Ny8lyREREREQDSZYjIiIiIhrIXzeITrP1zju+pf99EhEREbGl5WQ5IiIiIqKBJMsREREREQ2kDCM6zbpnn+GZ66/t6jD6rF3OG9/VIURERPQ6OVmOiIiIiGggyXJERERERANJliMiIiIiGugzybKksZIO6Oo4uoqkKyWd0M6YYyQd/lbFFBEREdHd9bpkWVK/BpfGAn02WbZ9ue2ftzPsGCDJckRERETRbZJlSZdIGl/a10i6r7SPl3RTaZ8haamkZZIm1MxdVU5O5wJjJF0labmkJZKuLqelJwETJS2SNKzV3rtKul3S4vJ1eOn/67LXMkkXlL6hkh6VdIOkRyRNl7Rduba3pJ+XNRZKGiZpoKR7y+ulkk4uYydI+quaGK6Q9DelfbGkeSX+Lzd4Xqsk/UNZ915JO5f+EZIeLHNvl/T20j9F0mmlvULSl2ti2l/SUOA84MLyjI6U9NFy74slzd6sH3BERERED9RtkmVgNnBkaTcBAyX1B44AmiXtDkwAjgNGAKMkjS3jBwDLbB8GLAdOAQ60fTDwVdtzgDuBi22PsP1Eq72vBWbZHg4cCjwiaSRwDnAY8D7gXEmHlPH7ANfZPhB4AfhI6Z9a+odTndA+BawGTrF9KHAs8A+SBNwMnF4Tw58Bt0g6saw/utznSElH1XleA4CFZd1ZwJdK/43A35Z7X1rT39pzZe63gItsrwCuB64pz6gZuBz4QLmfk+otImmcpPmS5v9+1aoGW0VERET0TN0pWV5AlRgOAtYAD1AlzUcCzcAoYKbtZ22vo0pMW5LI9cBtpb2SKkGdLOlU4JUO7H0cVdKI7fW2X6RK0m+3/bLtVcCPeD2Zf9L2opq4h5a497B9e1lnte1XAAF/J2kJ8HNgD2BX2w8Du0jaXdJw4Hnb/wWcWL4eBhYC+1Mlz61tAH5Y2jcBR0gaDAyxPav0f7/mGbX2o9r4G4y5H5gi6VygbnmL7Um2m2w3vWPgwAbLRERERPRM3eZDSWyvlbSC6jR3DrCE6iR2GPAosG8b01fbXl/WWSdpNHA88DHgs1TJ8MZSG9fW1LTXA9u1Mf5MYGdgZM09bluu3QqcBvwR1Ulzy75ft/3tjYzXGzm+5R7W0+Dfge3zJB0GfAhYJGmE7d9v5D4RERERPVZ3OlmGqhTjovK9maqGdpFtA3OBoyXtVN7EdwZV+cEbSBoIDLZ9N3ABVSkDwEvAoAb73gucX+b3k7RDiWGspO0lDaAq7WhuFLjtlcBvW0pDJG0jaXtgMPBMSZSPBd5ZM+1mqoT+NKrEGWAa8OlyH0jaQ9IudbbcqswD+Djwi3Ii/ryklhPws6jzjNrwhmckaZjtubYvB54D9tyItSIiIiJ6vO6WLDcDuwEP2H6aqpyiGcD2U8AXgBnAYqp63TvqrDEIuKuUPcwCLiz9NwMXS3q49Rv8gM8Dx0paSlWWcKDthcAU4CGqRH1yKZ1oy1nA+LL3HKoT46lAk6T5VKfMj7UMtv1Iife/y/1hezrwA+CBEs+t1E/yXwYOlLSA6uT8ytL/Kao3Mi6h+kXhyjpzG/l34JSWN/iVdZZKWkb1y8PijVgrIiIiosdTdWgbPY2kVba7VZHwiHfu5elfuKirw+izdjlvfFeHEBER0WNIWmC7qb1x3e1kOSIiIiKi20iy3EN1t1PliIiIiN6o2/w1jOj5tt55l5QCRERERK+Sk+WIiIiIiAaSLEdERERENJAyjOg0a5/5Db+77q+7Oow27f6Zf+zqECIiIqIHyclyREREREQDSZYjIiIiIhpIshwRERER0UCS5Y0kaYikv9qM+UMlfbwzY4qIiIiILSPJ8sYbAmxysgwMBTY6WZbUbzP2bL3W1m29fitiiIiIiOgJkixvvKuAYZIWSZoIIOliSfMkLZH05dI3qrzeVtIASY9IOqjMP7LMv1DS2ZL+uWVxSXdJOqa0V0m6UtJcYIykkZJmSVogaZqk3VoHJ2lnSbeVeOZJen/pv0LSJEnTgRvLvrdI+ndguioTJS2TtFTS6WXeMZJmSPoBsHRLPtiIiIiI7iZ/Om7jXQocZHsEgKQTgX2A0YCAOyUdZXu2pDuBrwLbATfZXibpUuAi2x8u889uY68BwDLbl0vqD8wCTrb9bElmvwZ8utWcbwDX2P6FpL2AacB7yrWRwBG2Xy37jgEOtv0HSR8BRgDDgZ2AeZJml3mjyz0/2TpASeOAcQB7vH1Q+08vIiIiogdJsrz5TixfD5fXA6mS59nAlcA8YDWwKZ8DvR64rbT3Aw4CfiYJoB/wVJ05JwAHlDEAO0hqyWLvtP1qzdif2f5DaR8B/Kvt9cDTkmYBo4CVwEP1EmUA25OASQDD99rVG3+LEREREd1XkuXNJ+Drtr9d59qOVMlzf2Bb4OU6Y9bxxnKYbWvaq0vy2rLPI7bHtBPPVsCYVkkxJXluvX/ta9FYvbgjIiIier3ULG+8l4DaeoNpwKclDQSQtIekXcq1ScAXganAhAbzVwAjJG0laU+qkod6fgnsLGlM2ae/pAPrjJsOfLblhaQRHbyv2cDpkvpJ2hk4Cniog3MjIiIieqWcLG8k27+XdL+kZcBPbV8s6T3AA+X0dhXwCUkfBNbZ/kH5KxJzJB0HNAPrJC0GpgD/BDxJ9ea5ZcDCBvu+Juk04FpJg6l+dv8EPNJq6HjgOklLypjZwHkduLXbqWqYFwMGLrH9fyXt37EnExEREdH7yE6ZaXSO4Xvt6p/+7ZldHUabdv/MP3Z1CBEREdENSFpgu6m9cSnDiIiIiIhoIMlyREREREQDqVmOTtN/lz1T5hARERG9Sk6WIyIiIiIaSLIcEREREdFAyjCi06x+5j947LqTu2z//T9zR5ftHREREb1TTpYjIiIiIhpIshwRERER0UCS5YiIiIiIBpIsdwOSxko6YGOvdXDtsyXtvunRRURERPRdSZbfQpL6Nbg0FmiUELd1rSPOBjYqWZaUN35GREREkGS5QyRdIml8aV8j6b7SPl7STaV9hqSlkpZJmlAzd5WkKyXNBcZIukrScklLJF0t6XDgJGCipEWShtXMfdO18nWPpAWSmiXtX8beIemTpf2XkqZKOg1oAqaW+dtJWiFppzKuSdLM0r5C0iRJ04EbJfWTNFHSvBLrX27hxxwRERHR7eQEsWNmA38DXEuVfG4jqT9wBNBcyhwmACOB54Hpksba/jEwAFhm+3JJOwLfAfa3bUlDbL8g6U7gLtu31m5qe07ra5LuBc6z/bikw4B/AY4DxgH3S3qyxPo+23+Q9FngItvzy/y27nMkcITtVyWNA160PUrSNmXt6baf3MxnGREREdFjJFnumAXASEmDgDXAQqqk+UhgPDAKmGn7WQBJU4GjgB8D64HbyjorgdXAZEk/Ae7amCAkDQQOB26pSXq3AbD9tKTLgRnAKbb/sAn3eaftV0v7RODgcjoNMBjYB3hDslyS6nEAu799u03YMiIiIqL7SrLcAbbXSloBnAPMAZYAxwLDgEeBfduYvtr2+rLOOkmjgeOBjwGfpToV7qitgBdsj2hw/b3A72m7Rnkdr5ffbNvq2ss1bQGfsz2trYBsTwImARy01xC3NTYiIiKip0nNcsfNBi4q35uB84BFtg3MBY6WtFN5E98ZwKzWC5ST4cG27wYuAFqS3peAQQ32/Z9rtlcCT0r6aFlPkoaX9mjgT4BDgIskvavB2iuoyi0APtLG/U4Dzi/lJkjaV9KANsZHRERE9DpJljuuGdgNeMD201TlFM0Atp8CvkBVArEYWGi73mcvDwLukrSEKpm+sPTfDFws6eHaN/g1uHYm8OeSFgOPACeXmuIbgE/b/h1VzfJ3VdVqTAGub3mDH/Bl4BuSmqlKRBqZDCwHFkpaBnyb/E9ERERE9DGqDkYjNt9Bew3xrX97dJftv/9n6v1+EhEREfFmkhbYbmpvXE6WIyIiIiIaSLIcEREREdFAalCj02y7y94phYiIiIheJSfLERERERENJFmOiIiIiGggZRjRaV5+9j94YNKHN3udMeM26oMNIyIiIraYnCxHRERERDSQZDkiIiIiooEkyxERERERDSRZ7kSSxko6oKvj6AhJQyT9VVfHEREREdGdJVneBJL6Nbg0FugRyTIwBKibLLdxfxERERF9Sp9KliVdIml8aV8j6b7SPl7STaV9hqSlkpZJmlAzd5WkKyXNBcZIukrScklLJF0t6XDgJGCipEWShrXa+6NlzcWSZpe+ZkkjasbcL+lgSVdI+r6k6ZJWSDpV0t+XuO6R1L+MXyHp7yQ9IGm+pEMlTZP0hKTzata9WNK8EuuXS/dVwLAS60RJx0iaIekHwFJJX5H0+Zo1vtby7CIiIiL6ij6VLAOzgSNLuwkYWBLPI4BmSbsDE4DjgBHAKEljy/gBwDLbhwHLgVOAA20fDHzV9hzgTuBi2yNsP9Fq78uBD9geTpVUA0wGzgaQtC+wje0l5dow4EPAycBNwAzb7wVeLf0tfmN7DNAMTAFOA94HXFnWPRHYBxhd7mmkpKOAS4EnSqwXl7VGA5fZPgD4DvCpssZWwMeAqe0/4oiIiIjeo68lywuoksVBwBrgAaqk+UiqZHMUMNP2s7bXUSWHR5W564HbSnslsBqYLOlU4JUO7H0/MEXSuUBLmcMtwIdLwv5pqmS3xU9trwWWlvH3lP6lwNCacXfW9M+1/ZLtZ4HVkoYAJ5avh4GFwP5UyXM9D9l+EsD2CuD3kg5pmW/7960nSBpXTrXnP7/qtQ48hoiIiIieo099KInttZJWAOcAc4AlwLFUp7iPAvu2MX217fVlnXWSRgPHU524fpbqNLqtvc+TdBjVqfAiSSNs/17Sz6hOj/+MKnFvsabM2yBprW2X/g288ee2pqZ/TU1/yzgBX7f97dp4JA2tE+bLrV63nHz/EfDdBvc1CZgE8J53DnG9MRERERE9VV87WYaqFOOi8r0ZOA9YVJLRucDRknYqb3I7A5jVegFJA4HBtu8GLqAqbwB4CRhUb1NJw2zPtX058BywZ7k0GbgWmGf7D510j7WmAZ8uMSNpD0m7tBVrjduBD1KduE/bArFFREREdGt96mS5aAYuAx6w/bKk1aUP209J+gIwg+pE9m7bd9RZYxBwh6Rty7gLS//NwA3ljXCntapbnihpnzL+XmBx2XOBpJXA9zr7Rsv60yW9B3hAEsAq4BO2nyhvKFwG/BT4SZ25r0maAbzQcqoeERER0Zfo9f/dj65Q3lQ4E9jf9oYuDucNyhv7FgIftf14e+Pf884h/u5lR2z2vmPG3bXZa0RERES0RdIC203tjeuLZRjdhqRPUpV+XNYNE+UDgP8A7u1IohwRERHRG/XFMoxuw/aNwI1dHUc9tpcD7+7qOCIiIiK6UpLl6DQDdt47JRQRERHRq6QMIyIiIiKigSTLERERERENJFmOiIiIiGggNcvRaVY+9zjTvvOnGzXnA39+9xaKJiIiImLz5WQ5IiIiIqKBJMsREREREQ0kWd4CJI0tH+rRrUnaXdKtpT1C0sbVUERERET0ckmWN4Okfg0ujQW6fbJs+3e2TysvRwBJliMiIiJq9MlkWdIlksaX9jWS7ivt4yXdVNpnSFoqaZmkCTVzV0m6UtJcYIykqyQtl7RE0tWSDgdOAiZKWiRpWKu9d5V0u6TF5evw0v/XZa9lki4ofUMlPSrpBkmPSJouabtybW9JPy9rLJQ0TNJASfeW10slnVzGTpD0VzUxXCHpb8r6yyS9DbgSOL3EfLqkxyXtXMZvJek/JO20hX4kEREREd1Sn0yWgdnAkaXdBAyU1B84AmiWtDswATiO6sR1lKSxZfwAYJntw4DlwCnAgbYPBr5qew5wJ3Cx7RG2n2i197XALNvDgUOBRySNBM4BDgPeB5wr6ZAyfh/gOtsHAi8AHyn9U0v/cOBw4ClgNXCK7UOBY4F/kCTgZuD0mhj+DLil5YXt14DLgR+WmH8I3AScWYacACy2/VyHnm5EREREL9FXk+UFwEhJg4A1wANUSfORQDMwCphp+1nb66gS06PK3PXAbaW9kipBnSzpVOCVDux9HPAtANvrbb9IlaTfbvtl26uAH/F6Mv+k7UU1cQ8tce9h+/ayzmrbrwAC/k7SEuDnwB7ArrYfBnYpNcrDgedt/1c7cX4X+GRpfxr4Xr1BksZJmi9p/osvvdaB24+IiIjoOfpksmx7LbCC6jR3DlWCfCwwDHiUKulsZLXt9WWddcBoquR5LHDPJobU1n5ratrrqf42dqPxZwI7AyNtjwCeBrYt124FTqM6Yb65vYBs/wZ4WtJxVCfeP20wbpLtJttNgwe9rb1lIyIiInqUPpksF7OBi8r3ZuA8YJFtA3OBoyXtVN7EdwYwq/UCkgYCg23fDVxAVbIB8BIwqMG+9wLnl/n9JO1QYhgraXtJA6hKO5obBW57JfDbltIQSdtI2h4YDDxje62kY4F31ky7GfgYVcJ8a51l68U8maoc499afkGIiIiI6Ev6crLcDOwGPGD7aapyimYA208BXwBmAIuBhbbvqLPGIOCuUvYwC7iw9N8MXCzp4dZv8AM+DxwraSlVWcWBthcCU4CHqBL1yaV0oi1nAePL3nOAP6IqF2mSNJ/qlPmxlsG2Hynx/ne5v9ZmAAe0vMGv9N0JDKRBCUZEREREb6fqIDXizSQ1AdfYPrLdwcC+Qwf7m198/0btkY+7joiIiK4gaYHtpvbGbf1WBBM9j6RLqcpFzmxvbERERERv1ZfLMKINtq+y/U7bv+jqWCIiIiK6Sk6Wo9PssNM+KauIiIiIXiUnyxERERERDSRZjoiIiIhoIMlyREREREQDqVmOTvP8c49z6/c+2O64087Z1A86jIiIiHhr5WQ5IiIiIqKBJMsREREREQ0kWe4jJE2WdEA7Y8a2NyYiIiKiL0my3EfY/gvby9sZNhZIshwRERFRdItkWdJQSY+V089lkqZKOkHS/ZIelzS6jBsg6buS5kl6WNLJNfObJS0sX4eX/mMkzZR0a1l/qiTV2X9vST+XtLjMH6bKxBLPUkmnt7empFGS5pR1HpI0qI3YfijpT2timCLpI5L6lX3nSVoi6S/beF7fL2NulbR9uXZ8eTZLy7PapvTPlNRU2qskfa3E+aCkXUtcJwETJS0qz2C8pOVlj5s782ceERER0RN0i2S52Bv4BnAwsD/wceAI4CLgf5cxlwH32R4FHEuV2A0AngH+2PahwOnAtTXrHgJcQHVi+m7g/XX2ngpcZ3s4cDjwFHAqMAIYDpxQ9tqt0ZqS3gb8EPh8WecE4NU2Yru5vKbMPR64G/hz4MVyj6OAcyW9q07M+wGTbB8MrAT+StK2wBTgdNvvpfprJ+fXmTsAeLDEORs41/Yc4E7gYtsjbD8BXAocUvY4r846EREREb1ad0qWn7S91PYG4BHgXtsGlgJDy5gTgUslLQJmAtsCewH9gRskLQVu4Y2lBA/Z/m1Zd1HNWgBIGgTsYft2ANurbb9Claj/q+31tp8GZlElr43W3A94yva8ss5K2+vaiO2nwHHl5PdPgNm2Xy33+Mlyj3OBdwD71Hlev7F9f2nfVOLdrzzHX5X+7wNH1Zn7GnBXaS9o/UxqLAGmSvoEsK7eAEnjJM2XNH/lqtcaLBMRERHRM3Wnv7O8pqa9oeb1Bl6PU8BHbP+ydqKkK4CnqU6BtwJWN1h3PW++5zeVZbTT32hNAa4z9sJ6sdleLWkm8AGqE+Z/rdn3c7antbE/dfZyOzHXWlt+EamNv54PUSXbJwFflHRg+QXg9U3tScAkgGFDB9e7/4iIiIgeqzudLHfENOBzNTXCh5T+wVSnuhuAs4B+HV3Q9krgt5LGljW3KfW/s4HTSw3xzlRJ40NtLPUYsLukUWWdQZK2bie2m4FzgCPLvbXc4/mS+pd19i2lJq3tJWlMaZ8B/KLEMFTS3qX/LKoT8Y56CRhU9t0K2NP2DOASYAgwcCPWioiIiOjxelqy/BWqsoYlkpaV1wD/AnxK0oPAvsDLG7nuWcB4SUuAOcAfAbdTlSEsBu4DLrH9fxstYPs1qhPib0paDPyMqkykrdimUyXhPy/zASYDy4GF5R6/Tf2T30fLukuAHYFv2V5NlXzfUso+NgDXb8RzuBm4WNLDVKUfN5V1Hgausf3CRqwVERER0ePp9f+Nj55C0lDgLtsHdXEobzBs6GBP+NKYdsfl464jIiKiq0laYLupvXE97WQ5IiIiIuIt053e4BcdZHsF0K1OlSMiIiJ6oyTL0WnevtM+KbGIiIiIXiVlGBERERERDSRZjoiIiIhoIMlyREREREQDqVmOTvPc73/F975/Yt1r53xq+lscTURERMTmy8lyREREREQDSZYjIiIiIhpIshwRERER0UCPTZYljZV0QINrO0uaK+lhSUdu5j5DJX28g+OWdWDcFEmnlfbkRvewJUk6T9In3+p9IyIiInqabp8sS+rX4NJYoFGieTzwmO1DbDd3cL1GhgLtJsubwvZf2F6+JdZuZ9/rbd/4Vu8bERER0dNssWRZ0iWSxpf2NZLuK+3jJd1U2mdIWippmaQJNXNXSbpS0lxgjKSrJC2XtETS1ZIOB04CJkpaJGlYzdwRwN8Df1qubVdnvcslzSv7TpKkMndvST+XtFjSwrLuVcCRZa0Lywlyc7m+sMTS1nOQpH8u8f8E2KXm2kxJTTX3PEHSghLD6HL915JOKmP6SZpYYl8i6S9L/zFl7K2SHpM0teae3vDsSt8Vki5qeV6SHizXb5f09prYJkh6SNKvNveEPiIiIqIn2pIny7OBlgSrCRgoqT9wBNAsaXdgAnAcMAIYJWlsGT8AWGb7MGA5cApwoO2Dga/angPcCVxse4TtJ1o2tb0IuBz4Ybn2au16tn8B/LPtUbYPArYDPlymTwWusz0cOBx4CrgUaC5rXQM8A/yx7UOB04Fr23kOpwD7Ae8Fzi3r1jMAmGl7JPAS8FXgj8v8K8uYPwdetD0KGAWcK+ld5dohwAVUp+3vBt4vacfWz67OvjcCf1uuLwW+VHNta9ujy7pfqjMXSeMkzZc0f9VLa9t+EhERERE9zJZMlhcAIyUNAtYAD1AlzUcCzVTJ3kzbz9peR5WoHlXmrgduK+2VwGpgsqRTgVc2IZba9QCOLTXNS6mS9QNLnHvYvh3A9mrb9fbqD9xQ5t5C41KQFkcB/2p7ve3fAfc1GPcacE9pLwVm2V5b2kNL/4nAJyUtAuYC7wD2Kdcesv1b2xuARWVOm89O0mBgiO1Zpev7vP4zAPhR+b6gJoY3sD3JdpPtpoGD+jd8CBERERE90RZLlkuitwI4B5hDlSAfCwwDHgXUxvTVtteXddYBo6mS3bG8nlBujP9ZT9K2wL8Ap9l+L3ADsG078dS6EHgaGE6V/L+tA3PcgTFrbbeM20D1CwYl+W358BgBnyun3CNsv8t2y6d9rKlZaz3VqfDmPruWNdeTD7CJiIiIPmhLv8FvNnBR+d4MnAcsKknhXOBoSTuVN92dAcxqvYCkgcBg23dTlQOMKJdeAgZtQkzblu/PlbVPA7BgBTpkAAAgAElEQVS9EvhtSymIpG0kbV9nn8HAUyWJPQto7w2Ds4GPlXrj3ah+YdhU04DzSzkLkvaVNKDR4DaeHQC2XwSer6lHPos6P4OIiIiIvmpLnxY2A5cBD9h+WdLq0oftpyR9AZhBdWJ6t+076qwxCLijnAiL6mQX4GaqcojxVKfET9SZ+ya2X5B0A1V5wwpgXs3ls4BvS7oSWAt8FFgCrJO0GJhCdSp9m6SPlthfbmfL26lKPZYCv2LzktHJVOUQC8sb+J6lOjFupNGzq/Up4Pryi8Gvqf4nICIiIiIAvf4//xGbZ+i7dvCXrnhf3WvnfGp63f6IiIiIriBpge2m9sZ1+7+zHBERERHRVfKmreg0O71j35wgR0RERK+Sk+WIiIiIiAaSLEdERERENJBkOSIiIiKigdQsR6d55g+Pc+3UD7ypf/yZ07ogmoiIiIjNl5PliIiIiIgGkixHRERERDSQZDkiIiIiooEkyz2IpAvKx1K3vF7VlfFERERE9HZJlnuWC4Dt2x0VEREREZ2ixyfLkoZKekzSZEnLJE2VdIKk+yU9Lml0GTdA0nclzZP0sKSTa+Y3S1pYvg4v/cdIminp1rL+VEmqs/94ScslLZF0c+m7QtL3JU2XtELSqZL+XtJSSfdI6l/GHV9iWVpi26ZRv6TxwO7ADEkzavb/mqTFkh6UtGvpmyLpWklzJP1a0mk14y8uz2CJpC/XPJuflHWWSTq99F9Vc29Xb4mfX0RERER31uOT5WJv4BvAwcD+wMeBI4CLgP9dxlwG3Gd7FHAsMFHSAOAZ4I9tHwqcDlxbs+4hVKe5BwDvBt5fZ+9LgUNsHwycV9M/DPgQcDJwEzDD9nuBV4EPSdoWmAKcXvq3Bs5v1G/7WuB3wLG2jy17DAAetD0cmA2cW7P/buUZfBi4CkDSicA+wGhgBDBS0lHAB4Hf2R5u+yDgHkk7AqcAB5Z7+2q9By9pnKT5kuavWvlavSERERERPVZvSZaftL3U9gbgEeBe2waWAkPLmBOBSyUtAmYC2wJ7Af2BGyQtBW6hSoxbPGT7t2XdRTVr1VoCTJX0CWBdTf9Pba8tMfQD7in9LTHtV+L+Ven/PnBUG/31vAbcVdoLWsX3Y9sbbC8Hdq15BicCDwMLqX6x2KfEdIKkCZKOtP0isBJYDUyWdCrwSr0AbE+y3WS7aeAOb2sQZkRERETP1Fs+lGRNTXtDzesNvH6PAj5i+5e1EyVdATwNDKf65WF1g3XXU/95fYgqmT0J+KKkA2vn2t4gaW1J3mtjelNJR02cHVW7buv4amNXzfev2/72mzaVRgJ/Cnxd0nTbV5YSluOBjwGfBY7biNgiIiIierzecrLcEdOAz7XUHUs6pPQPBp4qp8dnUZ0Cd4ikrYA9bc8ALgGGAAM7OP0xYKikvcvrs4BZbfQDvAQM6mh8dUwDPi1pYIl/D0m7SNodeMX2TcDVwKFlzGDbd1OVoozYjH0jIiIieqTecrLcEV8B/glYUhLmFVT1vP8C3Cbpo8AM4OWNWLMfcJOkwVSnttfYfqHO+wDfxPZqSecAt0jaGpgHXG97Tb3+Mm0S8FNJT9XULXeY7emS3gM8UGJcBXyCquZ7oqQNwFrgfKqk/I5SQy3gwo3dLyIiIqKn0+v/ix+xefZ692Bf9JX3val//JnTuiCaiIiIiMYkLbDd1N64vlSGERERERGxUfpSGUZsYbvsuE9OkSMiIqJXyclyREREREQDSZYjIiIiIhpIshwRERER0UBqlqPT/O75x7ni3z7whr4r/iw1zBEREdFz5WQ5IiIiIqKBJMsREREREQ0kWY6IiIiIaCDJcgdIGivpgG4Qx9mSdq95vULSTl0ZU0RERERvlmS5hqR+DS6NBbo8WQbOBnZvb1BEREREdI5ekSxLukTS+NK+RtJ9pX28pJtK+wxJSyUtkzShZu4qSVdKmguMkXSVpOWSlki6WtLhwEnAREmLJA1rtfdHy5qLJc0ufWdL+rGkf5f0pKTPSvprSQ9LelDSjmXciPJ6iaTbJb29Ub+k04AmYGqJY7sSwuckLSz3tn+Zf4Wk70qaKenXLc+mXPuEpIfKGt+W1K98TSn3sVTShWXs+JpncXPn/+QiIiIiurdekSwDs4EjS7sJGCipP3AE0FxKFyYAxwEjgFGSxpbxA4Bltg8DlgOnAAfaPhj4qu05wJ3AxbZH2H6i1d6XAx+wPZwqqW5xEPBxYDTwNeAV24cADwCfLGNuBP627LUU+FKjftu3AvOBM0scr5axz9k+FPgWcFHN/vsDHyj7f0lSf0nvAU4H3m97BLAeOLM8kz1sH2T7vcD3yhqXAoeUOM6r9+AljZM0X9L8V1a+Vm9IRERERI/VW5LlBcBISYOANVQJaRNVAt0MjAJm2n7W9jpgKnBUmbseuK20VwKrgcmSTgVe6cDe9wNTJJ0L1JZxzLD9ku1ngReBfy/9S4GhkgYDQ2zPKv3fB45q1N/G/j+qeQZDa/p/YnuN7eeAZ4BdgeOBkcA8SYvK63cDvwbeLembkj5YngPAEqqT7E8A6+ptbnuS7SbbTdvv8LY2woyIiIjoeXpFsmx7LbACOAeYQ5UgHwsMAx4F1Mb01bbXl3XWUZ3E3kZVp3xPB/Y+D/g/wJ7AIknvKJfW1AzbUPN6A537YTAt665vtW7t/i3XBHy/nEyPsL2f7StsPw8MB2YCnwEml3kfAq6jSrAXSMqH2ERERESf0iuS5WI2VRnCbKpk+TxgkW0Dc4GjJe1U3sR3BjCr9QKSBgKDbd8NXEBVngDwEjCo3qaShtmea/ty4DmqpLldtl8EnpfUUj5yFjCrUX97cXTQvcBpknYpse8o6Z3lL2psZfs24IvAoZK2Ava0PQO4BBgCDNyMvSMiIiJ6nN50UtgMXAY8YPtlSatLH7afkvQFYAbV6erdtu+os8Yg4A5J25ZxF5b+m4EbyhvlTmtVtzxR0j5l/L3AYl5PstvzKeB6SdtTlUKc007/lNL/KjCmg3v8D9vLJf0fYHpJhtdSnSS/Cnyv9AF8gaqk5KZSFiLgGtsvbOyeERERET2ZqoPXiM23+7DBHvf1972h74o/m9ZF0UREREQ0JmmB7ab2xvWmMoyIiIiIiE6VZDkiIiIiooHeVLMcXWz3t++TsouIiIjoVXKyHBERERHRQJLliIiIiIgGkixHp1nxwuOcc/sHuzqMiIiIiE6TZDkiIiIiooEkyxERERERDSRZjoiIiIhoIMlyREREREQDSZa7GUn93oI9tm71ukN7qpJ/MxEREdFnJPF5C0n6saQFkh6RNK6mf5WkKyXNBcZIGilpVhk7TdJuZdy5kuZJWizpNknb19ljgKTvlnEPSzq59J8t6RZJ/w5Ml3SMpBmSfgAsLWP+WtKy8nVB6Rsq6VFJ/wIsBPbc4g8qIiIioptIsvzW+rTtkUATMF7SO0r/AGCZ7cOAucA3gdPK2O8CXyvjfmR7lO3hwKPAn9fZ4zLgPtujgGOBiZIGlGtjgE/ZPq68Hg1cZvsASSOBc4DDgPcB50o6pIzbD7jR9iG2/7N2M0njJM2XNH/1ytc2/clEREREdEP5uOu31nhJp5T2nsA+wO+B9cBtpX8/4CDgZ5IA+gFPlWsHSfoqMAQYCNT7bOkTgZMkXVRebwvsVdo/s/2HmrEP2X6ytI8Abrf9MoCkHwFHAncC/2n7wXo3ZHsSMAlgp70Hu90nEBEREdGDJFl+i0g6BjgBGGP7FUkzqRJZgNW217cMBR6xPabOMlOAsbYXSzobOKbeVsBHbP+y1f6HAS+3Glv7Wm2E33peRERERJ+QMoy3zmDg+ZIo709V6lDPL4GdJY0BkNRf0oHl2iDgKUn9gTMbzJ8GfE7lWLqmlKI9s4GxkrYvZRunAM0dnBsRERHRKyVZfuvcA2wtaQnwFaBRWcNrwGnABEmLgUXA4eXyF6lqmn8GPNZgn68A/YElkpaV1+2yvZDq5Pqhssdk2w93ZG5EREREbyU7ZabROXbae7D/18QxfO+Ue7o6lIiIiIg2SVpgu6m9cTlZjoiIiIhoIMlyREREREQDSZaj0wwdsk9KMCIiIqJXSbIcEREREdFAkuWIiIiIiAaSLEenefyF/+JP7vhMV4cRERER0WmSLEdERERENJBkOSIiIiKigSTLERERERENJFneDJKGSvp4zeuzJf1zV8YUEREREZ0nyfLmGQp8vL1B3Y2krdt63dF5EREREb1dr0uWJQ2Q9BNJiyUtk3R66V8h6e8kPSBpvqRDJU2T9ISk88oYSZpY5i2tmVu3H7gKOFLSIkkXlr7dJd0j6XFJf18T1ypJXytxPShp19K/s6TbJM0rX+8v/UeXdRdJeljSIEm7SZpd+pZJOrLO/Y+UNEvSgnJ/u5X+meX+ZwGflzRF0j9KmgFMkLSjpB9LWlLiO7jMu0LSJEnTgRs7/ycWERER0X31xpPCDwK/s/0hAEmDa679xvYYSdcAU4D3A9sCjwDXA6cCI4DhwE7APEmzgcMb9F8KXGT7w2Wvs8u4Q4A1wC8lfdP2b4ABwIO2LytJ9LnAV4FvANfY/oWkvYBpwHuAi4DP2L5f0kBgNTAOmGb7a5L6AdvX3rik/sA3gZNtP1uS+q8Bny5Dhtg+uoydAuwLnGB7vaRvAg/bHivpOKrEeESZNxI4wvarG/ejiIiIiOjZemOyvBS4WtIE4C7bzTXX7qwZM9D2S8BLklZLGgIcAfyr7fXA0+UUdlQb/Svr7H+v7RcBJC0H3gn8BngNuKuMWQD8cWmfABwgqWX+DpIGAfcD/yhpKvAj27+VNA/4bkmKf2x7Uau99wMOAn5W1usHPFVz/Yetxt9S7olyjx8BsH2fpHfU/KJxZ6NEWdI4qiSebXceWG9IRERERI/V68owbP+K6iR0KfB1SZfXXF5Tvm+oabe83hoQ9TXqr6d23fW8/gvJWtuu078VMMb2iPK1h+2XbF8F/AWwHfCgpP1tzwaOAv4b+P8lfbJOnI/UrPVe2yfWXH+51fiXW81tzXXGvXGAPcl2k+2mt+2wXaNhERERET1Sr0uWJe0OvGL7JuBq4NCNmD4bOF1SP0k7UyWmD7XR/xIwaDNDng58tib+EeX7MNtLbU8A5gP7S3on8IztG4Dv1Lm3XwI7SxpT1ugv6cAOxjEbOLPMOwZ4zna9k/OIiIiIPqM3lmG8F5goaQOwFjh/I+beDowBFlOdql5i+/9KatT/e2CdpMVUNdDPb0K844HrJC2h+nnMBs4DLpB0LNUp9HLgp8DHgIslrQVWAW84Wbb9mqTTgGtLCcXWwD9R1WS35wrgeyWOV4BPbcK9RERERPQqer0yIGLzDN57Fx/+Dx/lpydf19WhRERERLRJ0gLbTe2N63VlGBERERERnSXJckREREREA0mWo9PsM2SvlGBEREREr5JkOSIiIiKigSTLERERERENJFmOiIiIiGggyXJERERERANJliMiIiIiGkiyHBERERHRQJLlTiZprKQDtuD6czppnWMkHd4Za0VERET0VkmWN5Gkfg0ujQU6PVlu2c92ZyW4xwAbtZakrTtp74iIiIgeoc8ly5IukTS+tK+RdF9pHy/pptI+Q9JSScskTaiZu0rSlZLmAmMkXSVpuaQlkq4uJ7UnARMlLZI0rNXeUyRdL6lZ0q8kfbj095M0UdK8stZflv5jJM2Q9ANgaUsMNddmSfq3stZVks6U9FCJfVgZt7Ok28ra8yS9X9JQ4DzgwhLnkfXGlflXSJokaTpw4xb6sURERER0S33xpHA28DfAtUATsI2k/sARQLOk3YEJwEjgeWC6pLG2fwwMAJbZvlzSjsB3gP1tW9IQ2y9IuhO4y/atDfYfChwNDANmSNob+CTwou1RkrYB7i/JKcBo4CDbT9ZZazjwHuAPwK+BybZHS/o88DngAuAbwDW2fyFpL2Ca7fdIuh5YZftqgJKQv2FcWZvyLI6w/epGPOeIiIiIHq8vJssLgJGSBgFrgIVUSfORwHhgFDDT9rMAkqYCRwE/BtYDt5V1VgKrgcmSfgLc1cH9/832BuBxSb8G9gdOBA6WdFoZMxjYB3gNeKhBogwwz/ZTJc4ngJYEeylwbGmfABwgqWXODuXeW2tr3J2NEmVJ44BxAHvttVfju46IiIjogfpcsmx7raQVwDnAHGAJVWI5DHgU2LeN6attry/rrJM0Gjge+BjwWeC4joRQ57WAz9meVntB0jHAy22staamvaHm9QZe/9luBYxpnezWJMV0YFzDGGxPAiYBNDU1tb63iIiIiB6tz9UsF7OBi8r3Zqr63UW2DcwFjpa0U3lT3RnArNYLSBoIDLZ9N1W5w4hy6SWg3slti49K2qrUFL8b+CVVycP5pRwESftKGtAJ9wnVafNna+JuFGejcRERERF9Vl9NlpuB3YAHbD9NVU7RDFDKGr4AzAAWAwv9/9i793i7qvLs+7+LEIlACAVSXvAxRkGkHANZgIHIyWirVoICIlAggEa0gqDIo9JaqtWKUKlAUSOFCKZIA6iACoGQE0FIds4Jxz6Crwrl8FLOJEByPX/MsV8Wi732XiHZ2afr+/nsz55zzDHucc+58se9R8Zay/5lBzGGAjdJWkpVTJ9Z2n8GfFnSosY3+BX3l/6/AU61vRK4DLgHWChpOfAj1t+q/+lArbxx8B6qPwwAbgQ+1v4Gv076RURERAxYqhZTY0OQNJnO3/zXp9VqNbe1tfV0GhERERFdkrTAdq2rfgN1ZTkiIiIioksD7g1+Pcn2hJ7OISIiIiJal5XliIiIiIgmUixHRERERDSRYjkiIiIiookUyxERERERTaRYjoiIiIhoIsVyREREREQTKZYjIiIiIppIsdyHSTpc0i49nUdEREREf5ViuQ+QNKjJpcOBN10sS8qX0kRERER0IsVyN5J0tqTTy/GFkm4vx++X9NNyfIykZZKWSzqvbuzzkr4h6W5gjKTvSLpH0lJJF0jaHzgMOF/SYkk7NMz9UUl3S1ok6TZJ25b2cyVNkjQNuFLSIEnnS5pfYn+m9Ntc0nRJC0t+4zfEM4uIiIjoTbKy2L1mA18CLgJqwCaSBgNjgTmStgfOA0YD/wNMk3S47V8AmwHLbX9d0lbAvwM727akLW0/LekG4Cbb13Yw9x3Ae0v/TwFnl1wo8421/ZKkicAztveRtAkwtxTSfwA+ZvtZSdsAd0m6wba740FFRERE9EZZWe5eC4DRkoYCq4DfUhXN7wPmAPsAM20/YftVYApwYBm7GriuHD8LrAQuk/Rx4MUW5v5fwC2SlgFfBnatu3aD7ZfK8QeBEyQtBu4GtgbeDQj4tqSlwG3A24BtGyeRNFFSm6S2J554ooW0IiIiIvqOFMvdyPYrwMPAScCdVAXyIcAOwL1UBWkzK22vLnFeBfalKp4PB25uYfqLgUts7w58BhhSd+2FumMBp9keVX7eaXsacBwwHBhtexTwWEOM9nucZLtmuzZ8+PAW0oqIiIjoO1Isd7/ZwFnl9xzgVGBx2c5wN3CQpG3Km/iOAWY1BpC0OTDM9q+BM4BR5dJzwNAm8w4D/lSOT+wkv1uAz5btIUjaSdJmZfzjtl+RdAjwjlZvOCIiIqK/SLHc/eYA2wG/tf0Y1XaKOQC2HwW+CswAlgALbf+ygxhDgZvKlohZwJml/WfAl8ub+HZoGHMuMFXSHODJTvK7DLgHWChpOfAjqr3sU4CapDaqVeb71uquIyIiIvoB5f1asb7UajW3tbX1dBoRERERXZK0wHatq35ZWY6IiIiIaCLFckREREREEymWIyIiIiKaSLEcEREREdFEiuWIiIiIiCZSLEdERERENJFiOSIiIiKiiS6LZUnbSvp3Sb8p57tIOqX7U4uIiIiI6FmtrCxPpvpK5O3L+QNUX7kcEREREdGvtVIsb2P7P4E1ALZfBVZ3a1YDlKTDJe3S03lERERERKWVYvkFSVsDBpD0XuCZbs1q4Doc6LBYlrTx+pqkMVarsSUNWl85RERERPQFrRTLXwRuAHaQNBe4EjitW7PaACRtJulXkpZIWi7paEnvl/Tzuj4fkHR9OX5e0nmSFki6TdK+kmZK+p2kw0qfCZJ+IelGSQ9J+rykL0paJOkuSVuVfjtIurnEmiNpZ0n7A4cB50taXPrMlPRtSbOAc0rMwSXGFpIebj+vy3m4pOskzS8/B5T2cyVNkjQNuLLkOlXSjcA0Vc4vz2KZpKPLuIMlzZD0H8Cy7n5dIiIiInqTTlcUJW0EDAEOAt4DCLjf9isbILfu9lfAI7Y/AiBpGPAs8G+Shtt+AjgJuKL03wyYaft/l4L6n4APUK0E/4TqDwqA3YC9qJ7bfwH/2/Zeki4ETgD+FZgEnGr7QUn7AZfaPlTSDcBNtq8tOQFsafugcj4S+AjwC+CTwHUdvBbfBy60fYekEVT7zf+iXBsNjLX9kqQJwBhgD9tPSToCGAXsCWwDzJc0u4zbF9jN9kNr/5gjIiIi+q5Oi2XbayT9i+0xwIoNlNOGsgy4QNJ5VAXqHABJVwF/I+kKqmLyhNL/ZeDmurGrbL8iaRkwsi7uDNvPAc9Jega4sW7MHpI2B/YHppZiGGCTTvK8pu74MuBsqmL5JODTHfQfB+xSF3sLSUPL8Q22X6rre6vtp8rxWOBq26uBx8pq9j5Uf0DMa1YoS5oITAQYMWJEJ7cRERER0fe0sld1Wll1vN62uzuhDcX2A5JGAx8G/lnSNNvfoFpJvhFYCUwtb2gEeKXu/tcAq0qcNQ17flfVHa+pO19D9bw3Ap62ParFVF+oy3mupJGSDgIG2V7eQf+NgDENRXH7KvULDX3rz0VzjeP+f7YnUa2UU6vV+s2/j4iIiAhofc/yVGCVpGclPSfp2W7Oq9tJ2h540fZPgQuAvQFsPwI8Avwd1cfmrVe2nwUeknRUyUOS9iyXnwOGNh1cuRK4mte2hzSaBny+/URSq0X5bOBoSYMkDQcOBOa1ODYiIiKiX+qyWLY91PZGtt9ie4tyvsWGSK6b7Q7Mk7QYOIdqD3K7KcAfbN/TTXMfB5wiaQnV9pbxpf1nwJfLGwJ3aDJ2CvBnVAVzR04HapKWSroHOLXFnH4OLAWWALcDZ9v+7xbHRkRERPRL6mpnhaQDO2q3Pbuj9v5A0iXAItv/3tO5NJJ0JDDe9vE9nUujWq3mtra2nk4jIiIiokuSFtiuddWvlT3LX647HkL1yQgLgEPfZG69mqQFVHt0v9TTuTSSdDHwIap91hERERHRzboslm1/tP5c0tuB73ZbRj3M9uiezqEZ233+860jIiIi+pJW3uDX6I9UnyUcEREREdGvdbmyXP7rv31j80ZUX1yxpDuTioiIiIjoDVrZs1z/jq1Xqb64Ym435RMRERER0Wu0Uixvafv79Q2SvtDYFhERERHR37SyZ/nEDtomrOc8IiIiIiJ6naYry5KOAY4F3inphrpLQ4H/r7sTi4iIiIjoaZ1tw7gTeBTYBviXuvbnqL7pLSIiIiKiX2taLNv+PfB7YMyGSyciIiIiovfocs+ypPdKmi/peUkvS1ot6dkNkdyGJulwSbv0dB5rq6/mHREREdHbtfIGv0uAY4AHgbcCnwIu7s6kupukQU0uHQ70xaKzr+YdERER0au19A1+tv8LGGR7te0rgEO6N62OSTpb0unl+EJJt5fj90v6aTk+RtIyScslnVc39nlJ35B0NzBG0nck3SNpqaQLJO0PHAacL2mxpB0a5t5W0s8lLSk/+5f2L5a5lks6o7SNlHSfpMtK+xRJ4yTNlfSgpH1Lv3MlXSXp9tL+6dK+uaTpkhaWexlfl8cJJeclZewb8pY0U9J5kuZJekDS+8rYQZLOL/9TsFTSZ0r7dpJml/HLJb2v9J1czpdJOrNbXtSIiIiIXqyVz1l+UdJbgMWSvkv1pr/NujetpmYDXwIuAmrAJpIGA2OBOZK2B84DRgP/A0yTdLjtX5Scl9v+uqStgH8HdrZtSVvafrp86sdNtq/tYO6LgFm2P1ZWpjeXNBo4CdgPEHC3pFll7h2Bo4CJwHyqTxYZS1XYfo1qNRhgD+C9Jb9Fkn4FPA58zPazkrYB7iq57QKcAxxg+0lJW9l+qjFvSQAb295X0oeBfwDGAacAz9jeR9ImwFxJ04CPA7fY/la5t02pvqnxbbZ3KzG37OgFkTSx3CMjRozo/NWLiIiI6GNaWVk+vvT7PPAC8HbgiO5MqhMLgNGShgKrgN9SFc3vA+YA+wAzbT9h+1VgCnBgGbsauK4cPwusBC6T9HHgxRbmPhT4AUBZYX+Gqvj9ue0XbD8PXF9yAXjI9jLba4AVwHTbBpYBI+vi/tL2S7afBGYA+1IV3t+WtBS4DXgbsG3J4drSF9tPdZLv9XXPrH2+DwInSFoM3A1sDbybqpg/SdK5wO62nwN+B7xL0sWS/qo8szewPcl2zXZt+PDhnaQTERER0fd0ubJs+/eS3gpsZ/sfN0BOneXyiqSHqVZz76T6CLtDgB2Ae4GdOhm+0vbqEufVshXi/cAnqf4QOPRNpKROrq2qO15Td76G1z93N4wzcBwwHBhdd89DynyN/buaf3XdfAJOs31LY2dJBwIfAa6SdL7tKyXtCfwl8LfAJ4CTW5w7IiIiol9o5dMwPgosBm4u56MavqRkQ5sNnFV+zwFOBRaXVdu7gYMkbVO2ExwDzGoMIGlzYJjtXwNnUG05gOozpIc2mXc68NkyfpCkLUoOh0vaVNJmwMdKTmtjvKQhkrYGDqZa5R0GPF4K5UOAd9Tl8InSl7KdpKu8690CfLZsXUHSTpI2k/SOMt+Pqban7F22f2xk+zrg74G91/K+IiIiIvq8VrZhnEu1NeBpANuLef02gg1tDrAd8Fvbj1Ftp5gDYPtR4KtU2xmWAAtt/7KDGEOBm8o2h1lA+5vXfgZ8WdKixjf4AV8ADpG0jGprw662F1jykaQAACAASURBVAKTgXlUhfplthet5f3MA34F3AV80/YjVNtHapLaqFaZ7yv3twL4FjBL0hLgey3kXe8y4B5goaTlwI+oVp0PptqTvohqi833qbZ+zCxbNiZTPdeIiIiIAUXVgmwnHaS7be8naZHtvUrbUtt7bJAM+7GyR/h52xf0dC7rQ61Wc1tbW0+nEREREdElSQts17rq18qnYSyXdCwwSNK7gdOp9gtHRERERPRrTbdhSLqqHP4fYFeqN4xdTfWpCGd0f2r9n+1z+8uqckRERER/1NnK8ujyxq+jqT5x4l/qrm1KtVc4IiIiIqLf6qxY/iHVJ2C8C6jfiNr+8WXv6sa8IiIiIiJ6XNNtGLYvsv0XwOW231X3807bKZQjIiIiot/r8qPjbH92QyQSEREREdHbtPI5yxERERERA1KK5YiIiIiIJlIsR0REREQ0scGKZUmnS7pX0pT1EGuCpO1b6DdZ0pFd9BlZvvoZSTVJF61rfm+GpHzRS0REREQv08o3+K0vnwM+ZPuh+kZJG9t+dS1jTQCWA4+sp9wAsN3G6z8mb4Oxvf+Gmqvxmbf6GrzJ1yoiIiKiz9ogxbKkH1J9LvMNki4HhgHbAyOBJyV9DbgK2KwM+bztO8vYs4HjgTXAb6iK2RowRdJLwBjgy8BHgbdSfRX3Z2y7k3xGA5cDLwJ31LUfDJxl+68lnQu8E9gO2An4IvBe4EPAn4CP2n6lxPoesDnwJDDB9qOSZgJ3U32hy5bAKbbnSNoVuAJ4C9XK/hG2H5T0vO3NJQn4bpnHwD/Zvqbkdm6ZYzdgAfA3jfcpaQfg34Dh5f4+bfs+SZOBp4C9gIWSnmt4DU4GflCe7avAF23PkDQB+AgwpLw+hzZ7rhERERH9zQbZhmH7VKpV4ENsX1iaRwPjbR8LPA58wPbeVN8YeBGApA8BhwP72d4T+K7ta6kK5uNsj7L9EnCJ7X1s70ZVMP91FyldAZxue0wX/XagKhTHAz8FZtjeHXgJ+IikwcDFwJG22wvwb9WN39j2vlRfD/4Ppe1U4Pu2R1EVpn9smPPjwChgT2AccL6k7cq1vUqsXaj++Digg5wnAaeVfM4CLq27thMwzvaXynn9a/C3AOX+jgF+ImlI6TcGONH2GwplSRMltUlqe+KJJzpIJyIiIqLv2pDbMBrdUApdgMHAJZJGAaupijqoisUrbL8IYPupJrEOKSvQmwJbASuAGzvqKGkYsKXtWaXpKqpV3I78pqweLwMGUX2jIcAyqhXZ91Ct8t5aLQgzCHi0bvz15feC0h/gt8A5kv4XcL3tBxvmHAtcbXs18JikWcA+wLPAPNt/LPexuMSsXxnfHNgfmFryAdikLvbUErdd/Wswlqrwp6xE/57XXodbmz1725OoCnRqtVrT1fyIiIiIvqgni+UX6o7PBB6jWk3dCFhZ2tu/Wrupsvp5KVCz/YeyfWJIZ0O6illnFYDtNZJeqdvysIbq2QlY0ckK9arye3Xpj+3/kHQ31Yr1LZI+Zfv2hvw6zacxZp2NgKfLqnVHXujkvLN5G8dFREREDAi95aPjhgGP2l5DtT95UGmfBpwsaVMASVuV9ueAoeW4vTB+sqysdvrpF7afBp6RNLY0HbcOed8PDJc0puQ3uOxJbkrSu4Df2b4IuAHYo6HLbOBoSYMkDQcOBOa1koztZ4GHJB1V5pKkPVu8l9mUZyFpJ2BEub+IiIiIAau3FMuXAidKuovqv/5fALB9M1VB2Va2HZxV+k8GfljaVgE/ptoa8QtgfgvznQT8m6TfUu0/flNsv0xVnJ8naQmwmGobRGeOBpaX3HcGrmy4/nNgKbAEuB042/Z/r0VaxwGnlHxWUO23bsWlwKCy5eQaqjcqrupiTERERES/pk4+NCJirdRqNbe19cgn70VERESsFUkLbNe66tdbVpYjIiIiInqdFMsREREREU2kWI6IiIiIaCLFckREREREEymWIyIiIiKaSLEcEREREdFEiuWIiIiIiCZSLEdERERENJFiOSIiIiKiiRTLA4ikhyVtU47v7Ol8IiIiInq7FMt9nKSN38w42/uv71wiIiIi+psUy2+CpJGS7pN0maTlkqZIGidprqQHJe1b+m0m6XJJ8yUtkjS+bvwcSQvLz/6l/WBJMyVdW+JPkaQO5p8p6duSZgFfkPRRSXeXOW6TtG3pt7WkaaX9R4DqYjxfN+dNde2XSJpQjr8j6R5JSyVd0G0PNCIiIqKXelOrkgHAjsBRwERgPnAsMBY4DPgacDhwDnC77ZMlbQnMk3Qb8DjwAdsrJb0buBqolbh7AbsCjwBzgQOAOzqYf0vbBwFI+jPgvbYt6VPA2cCXgH8A7rD9DUkfKbm2RNJWwMeAnUvcLZv0m9ged8SIEa2Gj4iIiOgTUiy/eQ/ZXgYgaQUwvRSVy4CRpc8HgcMknVXOhwAjqArhSySNAlYDO9XFnWf7jyXu4hKro2L5mrrj/wVcI2k74C3AQ6X9QODjALZ/Jel/1uL+ngVWApdJ+hVwU0edbE8CJgHUajWvRfyIiIiIXi/bMN68VXXHa+rO1/DaHyECjrA9qvyMsH0vcCbwGLAn1YryW5rEXU3zP2heqDu+GLjE9u7AZ6iK8nZdFbCv8vp/B0MAbL8K7AtcR7VKfnMXcSIiIiL6nRTL3esW4LT2fceS9irtw4BHba8BjgcGreM8w4A/leMT69pnA8eVuT8E/FkHY38P7CJpE0nDgPeX/psDw2z/GjgDGLWOOUZERET0OdmG0b2+CfwrsLQUzA8Dfw1cClwn6ShgBq9fJX4zzgWmSvoTcBfwztL+j8DVkhYCs4D/t3Gg7T9I+k9gKfAgsKhcGgr8UtIQqhXyM9cxx4iIiIg+R3a2mcb6UavV3NbW1tNpRERERHRJ0gLbta76ZRtGREREREQTKZYjIiIiIppIsRwRERER0USK5YiIiIiIJlIsR0REREQ0kWI5IiIiIqKJFMsREREREU2kWI6IiIiIaCLFckREREREEymWe4ik7SVd20K/r22IfCIiIiLijVIs9xDbj9g+soWu671YlrRxZ+etjouIiIjo73ptsSzpBElLJS2RdFVpe4ek6aV9uqQRpX2ypIsk3Snpd5KOrItztqRlJc53StunJc0vbddJ2lTSMEkPS9qo9NlU0h8kDZa0g6SbJS2QNEfSzh3ke66kqyTdLulBSZ8u7ZJ0vqTlJY+jS/tIScvL8QRJ15c5HpT03dL+HeCtkhZLmiJpM0m/Knkvb4/VkEeHuZZn9D1JM4DzSr6TJE0DrpQ0RNIVJcdFkg6py22qpBuBaevr9Y2IiIjoC3rlSqGkXYFzgANsPylpq3LpEuBK2z+RdDJwEXB4ubYdMBbYGbgBuFbSh8r1/Wy/WBfnets/LnP9E3CK7YslLQEOAmYAHwVusf2KpEnAqbYflLQfcClwaAep7wG8F9gMWCTpV8AYYBSwJ7ANMF/S7A7GjgL2AlYB90u62PZXJH3e9qiS6xHAI7Y/Us6HdRCns1x3AsbZXi3pXGA0MNb2S5K+BGB791JgT5O0Uxk3BtjD9lONk0maCEwEGDFiRAfpRERERPRdvXVl+VDgWttPAtQVaWOA/yjHV1EVx+1+YXuN7XuAbUvbOOAK2y82xNmtrLouA44Ddi3t1wDtq7WfBK6RtDmwPzBV0mLgR1SFeUd+afulkvcMYN+S49W2V9t+DJgF7NPB2Om2n7G9ErgHeEcHfZYB4ySdJ+l9tp+pv9hCrlNtr647v8H2S+V4LNUzxfZ9wO+pimuAWzsqlEvfSbZrtmvDhw/v+KlERERE9FG9cmUZEOAW+tX3WdUwvrM4k4HDbS+RNAE4uLTfAPxzWYEeDdxOtUr8dPvq7lrk036ujjp2oD7/1XTw2th+QNJo4MMlz2m2v1HXZaMucn2hk/PO8mwcFxERETEg9NaV5enAJyRtDVC3feJOqhVfqFaE7+gizjTgZEmbNsQZCjwqaXCJA4Dt54F5wPeBm8pq8LPAQ5KOKjEkac8m840ve3+3pirA5wOzgaMlDZI0HDiwzNGqV0qeSNoeeNH2T4ELgL3rO65lro1mU55F2X4xArh/LfKMiIiI6Hd6ZbFsewXwLWBW2Uf8vXLpdOAkSUuB44EvdBHnZqrV4rayLeGscunvgbuBW4H7GoZdA/xN+d3uOOCUkssKYHyTKecBvwLuAr5p+xHg58BSYAnVSvXZtv+7s7wbTAKWSpoC7A7MK/dyDvBPHfRvNddGlwKDytaUa4AJtld1MSYiIiKiX5Pdym6H6Ep5w9zzti/o6Vx6Sq1Wc1tbW0+nEREREdElSQts17rq1ytXliMiIiIieoPe+ga/Psf2uT2dQ0RERESsX1lZjoiIiIhoIsVyREREREQTKZYjIiIiIppIsRwRERER0USK5YiIiIiIJlIsR0REREQ0kWI5IiIiIqKJFMu9jKRvSBpXjs+QtGlP5xQRERExUKVY7mVsf932beX0DGC9F8uSBjWct/TlNK32i4iIiOgvUiwDkk6QtFTSEklXlbZ3SJpe2qdLGlHaJ0u6SNKdkn4n6ci6OGdLWlbifKe0fVrS/NJ2naRNJQ2T9LCkjUqfTSX9QdLgEv9ISacD2wMzJM2QdIqkC+vm+rSk73VwLx+U9FtJCyVNlbR5aX9Y0tcl3QEcJWmmpG9LmgV8oYv7/Z6kGcB53fQSRERERPRKA75YlrQrcA5wqO09gS+US5cAV9reA5gCXFQ3bDtgLPDXQHtR/CHgcGC/Eue7pe/1tvcpbfcCp9h+BlgCHFT6fBS4xfYr7RPYvgh4BDjE9iHAz4DDJA0uXU4Crmi4l22AvwPG2d4baAO+WNdlpe2xtn9Wzre0fZDtf+nifncqMb/U6cOMiIiI6GcGfLEMHApca/tJANtPlfYxwH+U46uoiuN2v7C9xvY9wLalbRxwhe0XG+LsJmmOpGXAccCupf0a4Ohy/Mly3pTtF4Dbgb+WtDMw2Payhm7vBXYB5kpaDJwIvKPueuMc9eed3e9U26s7ykvSREltktqeeOKJzm4hIiIios/JHlQQ4Bb61fdZ1TC+sziTgcNtL5E0ATi4tN8A/LOkrYDRVIVwVy4DvgbcR8Oqcl0Ot9o+psn4F7o4r1d/L0372Z4ETAKo1WqtPMeIiIiIPiMryzAd+ISkrQFK8QpwJ9WKL1Qrwnd0EWcacHL7p1fUxRkKPFq2TxzX3tn288A84PvATU1Wbp8r49vH3A28HTgWuLqD/ncBB0jaseSwqaSdusi73dreb0RERES/N+BXlm2vkPQtYJak1cAiYAJwOnC5pC8DT1DtEe4szs2SRgFtkl4Gfk21Cvz3wN3A74Fl1BW/VNsgpvLaanOjScBvJD1a9i0D/Ccwyvb/dJDDE2X1+mpJm5TmvwMe6Cz3Yq3uNyIiImIgkJ3/Oe9LJN0EXGh7ek/n0qhWq7mtra2n04iIiIjokqQFtmtd9cs2jD5C0paSHgBe6o2FckRERER/NOC3YfQVtp+m+gi3iIiIiNhAsrIcEREREdFEiuWIiIiIiCZSLEdERERENJFiOSIiIiKiiRTLERERERFNpFiOiIiIiGgixXJERERERBMplns5SRMkbb8O48+QtOn6zCkiIiJioEix3PtNAN50sQycAaxVsSwpX1YTERERQYrlLkkaKek+SZdJWi5piqRxkuZKelDSvqXfZpIulzRf0iJJ4+vGz5G0sPzsX9oPljRT0rUl/hRJapj7SKAGTJG0WNJbJY2WNEvSAkm3SNpO0sZl3oPLuH+W9C1Jp1MV2jMkzSjXnq+PL2lyOZ4s6Xul33nN7iciIiJiIMkKYmt2BI4CJgLzgWOBscBhwNeAw4FzgNttnyxpS2CepNuAx4EP2F4p6d3A1VQFMMBewK7AI8Bc4ADgjvZJbV8r6fPAWbbbJA0GLgbG235C0tHAt8qcE4BrS4H8V8B+tl+W9EXgENtPtnCfOwHjbK+W9O2O7sf2C2/uEUZERET0PSmWW/OQ7WUAklYA021b0jJgZOnzQeAwSWeV8yHACKpC+BJJo4DVVAVpu3m2/1jiLi6x7qC59wC7AbeWRehBwKMAtldIugq4ERhj++U3cZ9Tba/u4n7urR8gaSLVHxGMGDHiTUwZERER0XulWG7NqrrjNXXna3jtGQo4wvb99QMlnQs8BuxJte1lZZO4q+n69RCwwvaYJtd3B54Gtu0khuuOhzRcq1817vB+3hDMngRMAqjVau6sb0RERERfkz3L688twGnt+44l7VXahwGP2l4DHE+1Grw2ngOGluP7geGSxpQ5BkvatRx/HNgaOBC4qGydaBwP8Jikv5C0EfCxN3E/EREREQNGiuX155vAYGCppOXlHOBS4ERJd1FtwVjbPb+TgR+WbRqDgCOp3oC3BFgM7C9pG+A7wCm2HwAuAb5fxk8CftP+Bj/gK8BNwO2ULRxreT8RERERA4bs/M95rB+1Ws1tbW09nUZERERElyQtsF3rql9WliMiIiIimkixHBERERHRRIrliIiIiIgmUixHRERERDSRYjkiIiIiookUyxERERERTaRYjoiIiIhoIsVyREREREQTKZYjIiIiIppIsbyOJJ0q6YT1FOtr6yNORERERKwfKZbXgaSNbf/Q9pXrKeRaF8uSBr2JMRt3dt7quIiIiIj+bkAXP5JGAjcDdwN7AQ8AJ9h+UdJo4HvA5sCTwATbj0qaCdwJHADcIGko8LztC8q1RcBoYDhwAvBVYHfgGtt/V+b9G+B04C1l7s8B3wLeKmkxsML2cR31s71a0vMlt78EvgTcUXdPOwD/VuZ/Efi07fskTQaeKve5UNJzwPbASOBJSScDPwBqwKvAF23PkDQB+AgwBNgMOHTdnnpERERE35GVZXgPMMn2HsCzwOckDQYuBo60PRq4nKqYbbel7YNs/0sH8V62fSDwQ+CXwN8CuwETJG0t6S+Ao4EDbI8CVgPH2f4K8JLtUaVQ7rBfmWMzYLnt/Wzf8frpmQScVvI+C7i07tpOwDjbXyrno4Hxto8teWJ7d+AY4CeShpR+Y4ATbadQjoiIiAFlQK8sF3+wPbcc/5RqJfdmqgL3VkkAg4BH68Zc00m8G8rvZVQrxI8CSPod8HZgLFWROr/EfivweAdx3t9Jv9XAdY0DJG0O7A9MLWMANqnrMtX26vpcbb9UjsdS/YFAWYn+PVVxDXCr7ac6ullJE4GJACNGjOioS0RERESflWIZ3MG5qArdMU3GvNBJvFXl95q64/bzjUvsn9j+ahd5ddZvZUPR224j4OmyEt2Rxrzrz0VzTe/X9iSq1WxqtVrjs4yIiIjo07INA0ZIai+Kj6Ha/3s/MLy9XdJgSbuup/mmA0dK+vMSeytJ7yjXXilbQLrq1yHbzwIPSTqqjJGkPVvMazZlm4eknYARVM8hIiIiYsBKsQz3AidKWgpsBfzA9svAkcB5kpYAi6m2N6wz2/cAfwdMK3PeCmxXLk8Clkqa0kW/zhwHnFLyXgGMbzG1S4FBkpZRbTOZYHtVF2MiIiIi+jXZA/d/zsunYdxke7ceTqVfqNVqbmtr6+k0IiIiIrokaYHtWlf9srIcEREREdHEgH6Dn+2HqT71IiIiIiLiDbKyHBERERHRRIrliIiIiIgmUixHRERERDSRYjkiIiIiookUyxERERERTaRYjoiIiIhoIsVyREREREQTKZZ7kKSDJd1Ujg+T9JWezikiIiIiXjOgv5SkO0gS1deIr1mbcbZvAG7onqxeT9LGtl9tdt7quIiIiIj+LivL64GkkZLulXQpsBB4u6QfSGqTtELSP9b1/StJ90m6A/h4XfsESZeU48mSjqy79nz5vZ2k2ZIWS1ou6X0d5DJa0ixJCyTdImm70j5T0rclzQK+UOb4nqQZwHmStpL0C0lLJd0laY8y7lxJkyRNA67sjucXERER0VtlZXn9eQ9wku3PAUg6x/ZTkgYB00vx+QDwY+BQ4L+Aa9ZyjmOBW2x/q8TdtP6ipMHAxcB4209IOhr4FnBy6bKl7YNK38nATsA426slXQwssn24pEOpCuNRZdxoYKztl9Yy34iIiIg+LcXy+vN723fVnX9C0kSqZ7wdsAvVSv5Dth8EkPRTYOJazDEfuLwUxb+wvbjh+nuA3YBbq90gDAIerbveWJxPtb26HI8FjgCwfbukrSUNK9duaFYol3ucCDBixIi1uJWIiIiI3i/bMNafF9oPJL0TOAt4v+09gF8BQ8pltxDrVcprU/ZAvwXA9mzgQOBPwFWSTmgYJ2CF7VHlZ3fbH+woxw7O1UEe7qDf6zvYk2zXbNeGDx/e1X1FRERE9CkplrvHFlQF5jOStgU+VNrvA94paYdyfkyT8Q9TbX0AGA8MBpD0DuBx2z8G/h3Yu2Hc/cBwSWNK/8GSdm0x59nAcWXcwcCTtp9tcWxEREREv5RtGN3A9hJJi4AVwO+AuaV9Zdm28CtJTwJ3UG2baPRj4JeS5gHTeW1l92Dgy5JeAZ4HXreybPvl8sbAi8oWio2Bfy15dOVc4ApJS4EXgRNbv+OIiIiI/kl2K7sCIrpWq9Xc1tbW02lEREREdEnSAtu1rvplG0ZERERERBMpliMiIiIimkixHBERERHRRIrliIiIiIgmUixHRERERDSRYjkiIiIiookUyxERERERTaRYjoiIiIhoIsVyREREREQTKZYjIiIiIprol8WypJ0lLZa0SNIO6xhrlKQPt9DvYEk3tdBvpqRaOf61pC3XJb83Q9I3JI3b0PNGRERE9DX9slgGDgd+aXsv2/+nvVGVtb3nUUCXxfKbYfvDtp/ujthdzPt127dt6HkjIiIi+ppuKZYljZR0n6TLJC2XNEXSOElzJT0oad/SbzNJl0uaX1aBx9eNnyNpYfnZv7QfXFZmry3xp0hSw9wfBs4APiVpRol1r6RLgYXA2yX9QFKbpBWS/rFu7D6S7pS0RNI8ScOAbwBHl5XqoyXtW/osKr/f08WzeKukn0laKuka4K111x6WtM16eF4TJF0v6ebS/7ulfZCkySXmMklnlvbJko4sx+8vsZaV2JvU5faP5fkvk7Tzm/4HEREREdFHbdyNsXcEjgImAvOBY4GxwGHA16hWf88Bbrd9ctmOME/SbcDjwAdsr5T0buBqoFbi7gXsCjwCzAUOAO5on9T2ryX9EHje9gWSRgLvAU6y/TkASefYfkrSIGC6pD2A+4BrgKNtz5e0BfAi8HWgZvvzZewWwIG2Xy1bGb4NHNHJc/gs8KLtPco8C7vheUG1Ar4XsAq4X9LFwJ8Db7O9W8n9dVs+JA0BJgPvt/2ApCtLvv9aujxpe29JnwPOAj7VmLSkiSVnRowY0cljiIiIiOh7unMbxkO2l9leA6wApts2sAwYWfp8EPiKpMXATGAIMAIYDPxY0jJgKrBLXdx5tv9Y4i6ui9WZ39u+q+78E5IWAouoCu9dqArqR23PB7D9rO1XO4g1DJgqaTlwYRnfmQOBn5aYS4GlTfqty/Oi9H/G9krgHuAdwO+Ad0m6WNJfAc82zPmeMu8D5fwnJd9215ffC2jynG1Psl2zXRs+fHjzpxARERHRB3XnyvKquuM1dedr6uYVcITt++sHSjoXeAzYk6qgX9kk7mpau4cX6mK/k2qVdB/b/yNpMlXRKcAtxPomMMP2x8qq9cwWxrQSd12e13508FzK/e0J/CXwt8AngJPrh7aYU6vPOSIiIqJf6ek3+N0CnNa+71jSXqV9GNUq7xrgeGDQepxzC6ri+RlJ2wIfKu33AdtL2qfkMlTSxsBzwNC68cOAP5XjCS3MNxs4rsTcDdhjHXJv9rw6JGkbYCPb1wF/D+zd0OU+YKSkHcv58cCsdcgvIiIiol/p6WL5m1RbLpaWbQ3fLO2XAidKugvYibqV4XVlewnV9osVwOVU+56x/TJwNHCxpCXArVQrzjOAXdrf4Ad8F/hnSXNprYj/AbC5pKXA2cC8dUi/2fNq5m3AzLJtYzLw1fqLZcvGSVTbSpZRrWL/cB3yi4iIiOhXVG2LjVh3tVrNbW1tPZ1GRERERJckLbBd66pfT68sR0RERET0WimWIyIiIiKaSLEcEREREdFEiuWIiIiIiCZSLEdERERENJFiOSIiIiKiiRTLERERERFNpFiOiIiIiGgixXJERERERBMplgcoSSMlHdvTeURERET0ZimWB66RQIfFsqSNN2wqEREREb3TgCmWy0rqfZIuk7Rc0hRJ4yTNlfSgpH1Lv80kXS5pvqRFksbXjZ8jaWH52b+0HyxppqRrS/wpktTB/KdLukfSUkk/k7RRmXd4ub6RpP+StI2kyZJ+IGmGpN9JOqjkdK+kyXUxn5d0nqQFkm6TtG/J5XeSDit9Bkk6v9zPUkmfKcO/A7xP0mJJZ0qaIGmqpBuBaZKuar/3EmdKe8yIiIiIgWLAFMvFjsD3gT2AnalWVscCZwFfK33OAW63vQ9wCHC+pM2Ax4EP2N4bOBq4qC7uXsAZwC7Au4ADOpj7K8BetvcATrW9BvgpcFy5Pg5YYvvJcv5nwKHAmcCNwIXArsDukkaVPpsBM22PBp4D/gn4APAx4BulzynAM+V+9gE+LemdJZ85tkfZvrD0HQOcaPtQ4DLgJABJw4D9gV833pSkiZLaJLU98cQTHdx2RERERN810Irlh2wvK4XqCmC6bQPLqLYlAHwQ+IqkxcBMYAgwAhgM/FjSMmAqVWHcbp7tP5a4i+ti1VsKTJH0N8Crpe1y4IRyfDJwRV3/G+tye6wh7/b4LwM3l+NlwCzbr3RwPyeU+7kb2Bp4d5Pnc6vtpwBszwJ2lPTnwDHAdbZfbRxge5Ltmu3a8OHDm4SNiIiI6JsG2t7UVXXHa+rO1/DasxBwhO376wdKOhd4DNiT6o+MlU3irqbj5/oR4EDgMODvJe1q+w+SHpN0KLAfr60y18esz7MxLLTa4AAAGeVJREFU11dKQf26frbX1O07FnCa7Vsa7ufgDnJ8oeH8qpLTJ6mK+YiIiIgBZaCtLLfiFuC09n3HkvYq7cOAR8vq7vHAoFYDStoIeLvtGcDZwJbA5uXyZVTbMf7T9ur1cwuvcwvwWUmDSy47lW0lzwFDuxg7mWp7CbZXdENuEREREb1aiuU3+ibVloulkpaXc4BLgRMl3QXsxBtXYTszCPhp2cKxCLjQ9tPl2g1UhfMVzQavo8uAe4CF5X5+RLUyvRR4VdISSWd2NND2Y8C93ZhbRERERK+m1/4XP3qCpBpV8fy+ns6lkaRNqfY/7237ma7612o1t7W1dX9iEREREetI0gLbta76ZWW5B0n6CnAd8NWezqWRpHHAfcDFrRTKEREREf3RQHuDX69i+ztUn3fc69i+jepTQCIiIiIGrKwsR0REREQ0kWI5IiIiIqKJFMsREREREU2kWI6IiIiIaCLFckREREREEymWIyIiIiKaSLEcEREREdFEiuUBTFJN0kXl+GBJ+/d0ThERERG9Sb6UZACz3Qa0fz/1wcDzwJ09llBERERELzPgVpYljZR0n6TLJC2XNEXSOElzJT0oad/SbzNJl0uaL2mRpPF14+dIWlh+9i/tB0uaKenaEn+KJHUw/46SbpO0pIzfQZXzSz7LJB3dVUxJ+0i6s8SZJ2loJ7ldI+nDdTlMlnREiX+TpJHAqcCZkhZLep+khyQNLv23kPRw+3lERETEQDFQV5Z3BI4CJgLzgWOBscBhwNeAw4FzgNttnyxpS2CepNuAx4EP2F4p6d3A1UCtxN0L2BV4BJgLHADc0TD3FOA7tn8uaQjVHywfB0YBewLbAPMlzW4WU9I84BrgaNvzJW0BvNRJbj8DjgZ+LektwPuBzwL7Adh+WNIPgedtXwAgaSbwEeAXwCeB62y/0vggJU0sz5ERI/Lt2BEREdG/DLiV5eIh28tsrwFWANNtG1gGjCx9Pgh8RdJiYCYwBBgBDAZ+LGkZMBXYpS7uPNt/LHEX18UCQNJQ4G22fw5ge6XtF6kK9attr7b9GDAL2KeTmO8BHrU9v8R51varneT2G+BQSZsAHwJm236pi2d0GXBSOT4JuKKjTrYn2a7Zrg0fPryLkBERERF9y0BdWV5Vd7ym7nwNrz0TAUfYvr9+oKRzgceoVoE3AlY2ibuaNz7fN2zL6KK9WUwB7qDvmR3lVlaaZwJ/SbXCfHUn81HGzC3bOg4CBtle3tWYiIiIiP5moK4st+IW4LS6PcJ7lfZhVKu6a4DjgUGtBrT9LPBHSYeXmJtI2hSYDRwtaZCk4cCBwLxOQt0HbC9pnxJnqKSNu8jtZ1QrxO8r99boOWBoQ9uVVIV1h6vKEREREf1diuXmvkm1rWGppOXlHOBS4ERJdwE7AS+sZdzjgdMlLaX65In/B/g5sBRYAtwOnG37v5sFsP0y1QrxxZKWALdSbRPpLLdpVEX4bWV8oxuBj7W/wa+0TQH+jBZWoiMiIiL6I1VbdSPeSNKRwHjbx7fSv1arua2treuOERERET1M0gLbta76DdQ9y9EFSRdTvRnww131jYiIiOivUixHh2yf1tM5RERERPS07FmOiIiIiGgixXJERERERBMpliMiIiIimkixHBERERHRRIrliIiIiIgmUixHRERERDSRYjkiIiIiookUy/2EpAmStu/pPCIiIiL6kxTL/ccEIMVyRERExHrUp4plSSMl3SfpMknLJU2RNE7SXEkPStq39NtM0uWS5ktaJGl83fg5khaWn/1L+8GSZkq6tsSfIkkdzL+j/m97dx9ld1Xfe/z9ISAIRBClvX1wSAlwkWAeYMDLgxIxutraFbJqKBZEUlEWKg9q40Ol9lrRFksrikgxpTQ+5AISwQIuDRpCKMGQTCDJJEBEwK6bRS6o0YhgYk0+94/fjpwO85s5RzJzzsl8XmvNyvnts397f3/fOTPzzT77nCN9R9Kacv5EVS4v8fRLOqNhzKWSvirpe5Iuk3SWpBWl38TSb76ka0pc35P0J0PFWu77YBljTRl3NtALLJC0WtKLJf1A0t+Wc/slHTlMbiaV2FZLWivp8NL3G2WedTuvLSIiImKs6MaPuz4MOB04D1gJnAmcDMwEPgLMAi4B7rT9dkkHAiskfQd4CniD7a2SDgeupyoyAaYBk4AngGXAScA9A+ZeAFxm+xZJ+1D9Z+NPganAFODlwEpJd5f+U4BXApuBx4BrbR8v6WLgQuC9pd8E4BRgIrBE0mF1sUr6o3KNr7b9rKSDbG+WdAEw13YfQKn1f2T7GEnvBuYC7xgiN+cDn7W9QNKLgHHAHwNP2H5TGfOAgd8MSeeV7wU9PT2137SIiIiIbtRVK8vF47b7be8A1gOLbRvopyo6Ad4IfFjSauAuYB+gB9gL+BdJ/cBNwFEN466wvbGMu7phLAAkjQd+z/YtALa32n6WqlC/3vZ2208CS4HjymkrbW+yvQ14FLijtPcPGP+rtnfYfoSqqD5yiFhnAP9W5sb25iFydXP5d1UTufku8BFJHwIOsf2LEucMSZ+S9BrbWwZOYHue7V7bvQcffPAQoURERER0n25cWd7WcHtHw/EOnrseAW+2vaHxREkfA56kWvHdA9haM+52np+b523LGKa92VgBPOA8A++riVWD9B9u/sbrGTQ3wEOS7gPeBCyS9A7bd0o6lmqF+e8l3WH7403OHREREdH1unFluRmLgAt37juWNK20HwBsKqvHZ1NtNWiK7Z8BGyXNKmPuLWlf4G7gDEnjJB0MvBZY0WK8p0vao+xjPhTYMESsdwBvL3Mj6aDS/jQwvom5Bs2NpEOBx2xfCdwKTFb17hrP2v4K8I/AMS1eV0RERERX212L5UuptjGslbSuHANcDZwjaTlwBPBMi+OeDVwkaS1wL/A/gFuAtcAa4E7gg7b/X4vjbqDavvFN4HzbW+titf0tqmK2r2ylmFvGmA9cs/MFfkPMVZebM4B1ZcwjgS8Br6La07yaaq/zJ1q8roiIiIiupmq7b7SLpPnA7bYXtjuWF6q3t9d9fX3tDiMiIiJiWJJW2e4drt/uurIcEREREfGCdeML/HYrtue0O4aIiIiIGFxWliMiIiIiaqRYjoiIiIiokWI5IiIiIqJGiuWIiIiIiBopliMiIiIiaqRYjoiIiIiokWI5IiIiIqJGiuWIiIiIiBoplscwSflQmoiIiIghpFgeIZImSHpY0rWS1klaIGmGpGWSHpF0fOm3n6TrJK2U9ICk0xrO/w9J95evE0v7dEl3SVpYxl8gSYPM/84y5hpJX5O0b2mfL+nTkpYAn2p1/oiIiIixJMXyyDoM+CwwGTgSOBM4GZgLfKT0uQS40/ZxwOuAyyXtBzwFvMH2McAZwJUN404D3gscBRwKnDTI3DfbPs72FOAh4NyG+44AZtj+y99w/oiIiIgxIU/Dj6zHbfcDSFoPLLZtSf3AhNLnjcBMSXPL8T5AD/AEcJWkqcB2qgJ3pxW2N5ZxV5ex7hkw99GSPgEcCOwPLGq47ybb21/A/L8m6TzgPICenp5hExIRERHRTVIsj6xtDbd3NBzv4LncC3iz7Q2NJ0r6GPAkMIXqGYCtNeNuZ/Dv43xglu01kuYA0xvue6Zxqt9g/l+zPQ+YB9Db2+vB+kRERER0q2zDaL9FwIU79x1LmlbaDwA22d4BnA2Ma3Hc8cAmSXsBZ7Vh/oiIiIiul2K5/S4F9gLWSlpXjgGuBs6RtJxqC8QzNefX+ShwH/Bt4OE2zB8RERHR9WTnmfPYNXp7e93X19fuMCIiIiKGJWmV7d7h+mVlOSIiIiKiRorliIiIiIgaKZYjIiIiImqkWI6IiIiIqJFiOSIiIiKiRorliIiIiIgaKZYjIiIiImqkWI6IiIiIqJFiOSIiIiKiRorlLiLpdyUt3EVjzZJ01K4YKyIiImJ3lWK5S0ja0/YTtmfvoiFnAS0Vy5L23EVzR0RERHSFMVUsS5og6WFJ10paJ2mBpBmSlkl6RNLxpd9+kq6TtFLSA5JOazj/PyTdX75OLO3TJd0laWEZf4EkDTL/XZI+I+neMv9w882RdJOk24A7yvzrGu77uqTbJD0u6QJJ7y/nL5d0UOk3UdK3JK0qsR9Z4p4JXC5pdenzvH7l/PmSPi1pCfCpkf4eRURERHSSsbhSeBhwOnAesBI4EziZqnj8CNWK6yXAnbbfLulAYIWk7wBPAW+wvVXS4cD1QG8ZdxowCXgCWAacBNwzyPz72T5R0muB64Cjh5gP4ARgsu3NkiYMGOvoMu8+wPeBD9meJukK4G3AZ4B5wPm2H5H0auBq26dKuhW43fZCAEmLB/YDTi3zHAHMsL296SxHRERE7AbGYrH8uO1+AEnrgcW2LakfmFD6vBGYKWluOd4H6KEqhK+SNBXYTlVE7rTC9sYy7uoy1mDF8vUAtu+W9JJSHNfNB/Bt25trrmWJ7aeBpyVtAW4r7f3AZEn7AycCNzUsdO89cJAm+t1UVyhLOo/qPx709PQM1iUiIiKia43FYnlbw+0dDcc7eC4fAt5se0PjiZI+BjwJTKHawrK1Ztzt1OfWgxzXzfdq4JkXcC17AD+1PXWIMWiiX20MtudRrV7T29s78NoiIiIiutqY2rPcgkXAhTv3HUuaVtoPADbZ3gGcDYz7DcY+o4x5MrDF9pYh5ntBbP8MeFzS6WVcSZpS7n4aGN9Ev4iIiIgxK8Xy4C4F9gLWlhfUXVrarwbOkbScagvGUKu+dX4i6V7gGuDcYebbFc4CzpW0BlgPnFbabwA+UF4QOHGIfhERERFjluw8cz5aJN0FzLXd1+5YRkJvb6/7+nbLS4uIiIjdjKRVtnuH65eV5YiIiIiIGmPxBX5tY3t6u2OIiIiIiOZlZTkiIiIiokaK5YiIiIiIGimWIyIiIiJqpFiOiIiIiKiRYjkiIiIiokaK5YiIiIiIGimWIyIiIiJqpFgeAZIOlPTudscRERERES9MiuWRcSDQscWypHFDHQ9xXj7EJiIiIsaUri6WJb1N0lpJayR9ubQdImlxaV8sqae0z5f0z5KWSHpM0imSrpP0kKT5DWP+XNI/Sbq/nH9waX+npJVlrq9J2re0/7akW0r7GkknApcBEyWtlnS5pOmS7pK0UNLDkhZIUjn/WElLJa2StEjS75T2iyQ9WK7jhtJ2ShlztaQHJI0fJCdvlbSi9PnCzkK4XNfHJd0HnCDpB5L+RtI9wOmSpkpaXua7RdJLy3l3Sfo7SUuBi0fmOxkRERHRmbq2WJY0CbgEONX2FJ4r5K4CvmR7MrAAuLLhtJcCpwLvA24DrgAmAa+SNLX02Q+43/YxwFLgf5f2m20fV+Z6CDi3tF8JLC3txwDrgQ8Dj9qeavsDpd804L3AUcChwEmS9gI+B8y2fSxwHfDJ0v/DwLRyHeeXtrnAe2xPBV4D/GJATl4JnAGcVPpsB85quK51tl9t+57SttX2ybZvAL4EfKjM199w3QAH2j7F9j8RERERMYZ089PqpwILbf8IwPbm0n4C8Kfl9peBf2g45zbbltQPPGm7H0DSemACsBrYAdxY+n8FuLncPlrSJ6i2WOwPLGqI420lhu3Alp2rsgOssL2xzLe6zPdT4Gjg22WheRywqfRfCyyQ9HXg66VtGfBpSQuoiveNA+Z4PXAssLKM92LgqXLfduBrA/rfWOI5gKogXlravwjcNLDfYCSdB5wH0NPTU9ctIiIioit1c7EswE30a+yzrfy7o+H2zuO6XOw8fz4wy/YaSXOA6c0GOmBuqArXPamuYb3tEwbp/ybgtcBM4KOSJtm+TNI3gD8GlkuaYfvhhnMEfNH2Xw0y3tZSzDd6psnYa/vZngfMA+jt7W3m+xERERHRNbp2GwawGPgzSS8DkHRQab8XeEu5fRZwzyDnDmUPYHa5fWbD+eOBTWXrxFkN/RcD7yoxjJP0EuDp0n84G4CDJZ1Qzt9L0iRJewCvsL0E+CBlNVvSRNv9tj8F9AFHDhhvMTBb0m+V8Q6SdMhwQdjeAvxE0mtK09lUW1AiIiIixrSuXVm2vV7SJ4GlkrYDDwBzgIuA6yR9APgh8BctDv0MMEnSKmAL1R5ggI8C9wH/SbWnd2cxfDEwT9K5VCvG77L9XUnLJK0Dvgl8o+YafilpNnBl2QqxJ/AZ4HvAV0qbgCts/1TSpZJeV+Z5sIzdON6Dkv4auKMU3P8FvKfEPJxzgGvKCxcfo/W8RUREROx2ZOeZ80aSfm57/3bH0Y16e3vd19fX7jAiIiIihiVple3e4fp18zaMiIiIiIgRlWJ5gKwqR0RERMROKZYjIiIiImqkWI6IiIiIqJEX+MUuI+lpqrfDi+a8HPhRu4PoMslZa5Kv1iRfrUvOWpN8tW4kc3aI7YOH69S1bx0XHWlDM68qjYqkvuSrNclZa5Kv1iRfrUvOWpN8ta4TcpZtGBERERERNVIsR0RERETUSLEcu9K8dgfQZZKv1iVnrUm+WpN8tS45a03y1bq25ywv8IuIiIiIqJGV5YiIiIiIGimWo2WS/lDSBknfl/ThQe7fW9KN5f77JE0Y/Sg7RxP5eq2k+yX9StLsdsTYSZrI1/slPShpraTFkg5pR5ydpImcnS+pX9JqSfdIOqodcXaK4fLV0G+2JEsa8+9e0MRjbI6kH5bH2GpJ72hHnJ2imceYpD8rv8vWS/o/ox1jJ2ni8XVFw2Pre5J+OqoB2s5Xvpr+AsYBjwKHAi8C1gBHDejzbuCacvstwI3tjrvD8zUBmAx8CZjd7pi7IF+vA/Ytt981lh9fLeTsJQ23ZwLfanfcnZyv0m88cDewHOhtd9ydnjNgDnBVu2PthK8m83U48ADw0nL8W+2Ou5PzNaD/hcB1oxljVpajVccD37f9mO1fAjcApw3ocxrwxXJ7IfB6SRrFGDvJsPmy/QPba4Ed7QiwwzSTryW2ny2Hy4HfH+UYO00zOftZw+F+wFh+sUozv8MALgX+Adg6msF1qGZzFpVm8vVO4PO2fwJg+6lRjrGTtPr4+nPg+lGJrEixHK36PeD/NhxvLG2D9rH9K2AL8LJRia7zNJOveE6r+ToX+OaIRtT5msqZpPdIepSqALxolGLrRMPmS9I04BW2bx/NwDpYsz+Xby7boxZKesXohNaRmsnXEcARkpZJWi7pD0ctus7T9O/9su3uD4A7RyGuX0uxHK0abIV44CpVM33GiuSiNU3nS9JbgV7g8hGNqPM1lTPbn7c9EfgQ8NcjHlXnGjJfkvYArgD+ctQi6nzNPMZuAybYngx8h+eeXRyLmsnXnlRbMaZTrZReK+nAEY6rU7Xyd/ItwELb20cwnudJsRyt2gg0rhj8PvBEXR9JewIHAJtHJbrO00y+4jlN5UvSDOASYKbtbaMUW6dq9TF2AzBrRCPqbMPlazxwNHCXpB8A/wu4dYy/yG/Yx5jtHzf8LP4LcOwoxdaJmv07+e+2/8v248AGquJ5LGrld9hbGOUtGJBiOVq3Ejhc0h9IehHVA/fWAX1uBc4pt2cDd7rsyh+DmslXPGfYfJWnyL9AVSiP5X1+OzWTs8Y/wm8CHhnF+DrNkPmyvcX2y21PsD2Bal/8TNt97Qm3IzTzGPudhsOZwEOjGF+naeb3/tepXqyMpJdTbct4bFSj7BxN/Z2U9D+BlwLfHeX4UixHa8oe5AuARVS/DL9qe72kj0uaWbr9K/AySd8H3g/UvjXT7q6ZfEk6TtJG4HTgC5LWty/i9mry8XU5sD9wU3kboTH9n48mc3ZBeXuq1VQ/k+fUDLfbazJf0aDJnF1UHmNrqPbEz2lPtO3XZL4WAT+W9CCwBPiA7R+3J+L2auFn8s+BG9qx+JZP8IuIiIiIqJGV5YiIiIiIGimWIyIiIiJqpFiOiIiIiKiRYjkiIiIiokaK5YiIiIiIGimWIyKiJZLuHeX5Jkg6czTnjIjYKcVyRES0xPaJozVX+RTQCUCK5Yhoi7zPckREtETSz23vL2k68LfAk8BU4GagH7gYeDEwy/ajkuYDW4FJwG8D77d9u6R9gH8GeoFflfYlkuZQfdLgPsB+wL7AK4HHgS8CtwBfLvcBXGD73hLPx4AfUX1k9SrgrbYt6Tjgs+WcbcDrgWeBy4DpwN7A521/YRenKyK63J7tDiAiIrraFKpCdjPVx/Vea/t4SRcDFwLvLf0mAKcAE4Elkg4D3gNg+1WSjgTukHRE6X8CMNn25lIEz7X9JwCS9gXeYHtr+Sjv66kKboBpVEX5E8Ay4CRJK4AbgTNsr5T0EuAXwLnAFtvHSdobWCbpDtuPj0CeIqJLpViOiIgXYqXtTQCSHgXuKO39wOsa+n3V9g7gEUmPAUcCJwOfA7D9sKT/BHYWy9+2vblmzr2AqyRNBbY3nAOwwvbGEs9qqiJ9C7DJ9soy18/K/W8EJkuaXc49ADicagU7IgJIsRwRES/MtobbOxqOd/Df/8YM3PNnQEOM+8wQ972PauvHFKrX3mytiWd7iUGDzE9pv9D2oiHmiogxLi/wi4iI0XC6pD0kTQQOBTYAdwNnAZTtFz2lfaCngfENxwdQrRTvAM4Gxg0z98PA75Z9y0gaX144uAh4l6S9dsYgab8hxomIMSgryxERMRo2AEupXuB3ftlvfDVwjaR+qhf4zbG9TXregvNa4FeS1gDzgauBr0k6HVjC0KvQ2P6lpDOAz0l6MdV+5RnAtVTbNO5XNekPgVm74mIjYveRd8OIiIgRVd4N43bbC9sdS0REq7INIyIiIiKiRlaWIyIiIiJqZGU5IiIiIqJGiuWIiIiIiBopliMiIiIiaqRYjoiIiIiokWI5IiIiIqJGiuWIiIiIiBr/H4qXj5itwoNWAAAAAElFTkSuQmCC
"/>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</body>