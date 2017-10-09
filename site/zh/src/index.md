---
layout: default
body_class: body--index

logos:
- title: APEX
  image_url: /images/logo_apex.png
  url: "http://apex.apache.org"
- title: Flink
  image_url: /images/logo_flink.png
  url: "http://flink.apache.org"
- title: Spark
  image_url: /images/logo_spark.png
  url: http://spark.apache.org/
- title: Google Cloud Dataflow
  image_url: /images/logo_google_cloud.png
  url: https://cloud.google.com/dataflow/
- title: Gearpump
  image_url: /images/logo_gearpump.png
  url: http://gearpump.apache.org/  

pillars:
- title: Unified（统一性）
  body: 针对批处理和流式处理, 都使用了单个的编程模型.
- title: Portable（可移植性）
  body: 在多个执行环境中执行 pipelines（管道）.
- title: Extensible（可扩展性）
  body: Write 和共享新的 SDKs, IO connectors 以及 transformation libraries.

cards:
- quote: "提供了客户所需的灵活性以及一些高级功能的框架."
  name: –Talend
- quote: "Apache Beam 有强大的语义支持, 它解决了流式处理工作上的挑战."
  name: –PayPal
- quote: "Apache Beam 代表分析数据流的原则性方法."
  name: –data Artisans
---
<div class="hero-bg">
  <div class="hero section">
    <div class="hero__cols">
      <div class="hero__cols__col">
        <div class="hero__cols__col__content">
          <div class="hero__title">
            Apache Beam: 一个高级且统一的编程模型
          </div>
          <div class="hero__subtitle">
            让批处理和流数据处理的作业在任何执行引擎上都可以运行.
          </div>
          <div class="hero__ctas hero__ctas--first">
            <a class="button button--primary" href="{{'/get-started/beam-overview/'|prepend:site.baseurl}}">了解更多</a>
          </div>
          <div class="hero__ctas">
            <a class="button" href="{{'/get-started/quickstart-java/'|prepend:site.baseurl}}">Java 快速入门</a>
            <a class="button" href="{{'/get-started/quickstart-py/'|prepend:site.baseurl}}">Python 快速入门</a>
          </div>
        </div>
      </div>
      <div class="hero__cols__col">
        <div class="hero__blog">
          <div class="hero__blog__title">
            最新的博客
          </div>
          <div class="hero__blog__cards">
            {% for post in site.posts limit:3 %}
            <a class="hero__blog__cards__card" href="{{ post.url | prepend: site.baseurl }}">
              <div class="hero__blog__cards__card__title">{{post.title}}</div>
              <div class="hero__blog__cards__card__date">{{ post.date | date: "%b %-d, %Y" }}</div>
            </a>
            {% endfor %}
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

<div class="pillars section">
  <div class="pillars__title">
    关于 Apache Beam
  </div>
  <div class="pillars__cols">
    {% for pillar in page.pillars %}
    <div class="pillars__cols__col">
      <div class="pillars__cols__col__title">
        {{pillar.title}}
      </div>
      <div class="pillars__cols__col__body">
        {{pillar.body}}
      </div>
    </div>
    {% endfor %}
  </div>
</div>

<div class="graphic section">
<div class="graphic__image">
<img src="{{ '/images/beam_architecture.png' | prepend: site.baseurl }}" alt="Beam architecture">
</div>
</div>

<div class="logos section">
  <div class="logos__title">
    适用于
  </div>
  <div class="logos__logos">
    {% for logo in page.logos %}
    <div class="logos__logos__logo">
      <a href="{{ logo.url | prepend: base.siteUrl }}"><img src="{{logo.image_url|prepend:site.baseurl}}" alt="{{logo.title}}"></a>
    </div>
    {% endfor %}
  </div>
</div>

<div class="cards section section--wide">
  <div class="section__contained">
    <div class="cards__title">
      客户评价
    </div>
    <div class="cards__cards">
      {% for card in page.cards %}
      <div class="cards__cards__card">
        <div class="cards__cards__card__body">
          {{card.quote}}
        </div>
        <div class="cards__cards__card__user">
          <!-- TODO: Implement icons.
          <div class="cards__cards__card__user__icon">
          </div>
          -->
          <div class="cards__cards__card__user__name">
            {{card.name}}
          </div>
        </div>
      </div>
      {% endfor %}
    </div>
    <div class="cards__body">
      Beam 是一个开源社区,  非常感谢大家的贡献!
      如果您想要参与贡献, 请参阅 <a href="{{'/contribute/'|prepend:site.baseurl}}">贡献</a> 部分.
    </div>
  </div>
</div>

<div class="ctas section">
  <div class="ctas__title">
    入门指南
  </div>
  <div class="ctas__ctas ctas__ctas--top">
  <a class="button button--primary" href="{{'/get-started/beam-overview/'|prepend:site.baseurl}}">了解更多</a>
  </div>
  <div class="ctas__ctas">
  <a class="button" href="{{'/get-started/quickstart-java/'|prepend:site.baseurl}}">Java 快速入门</a>
  <a class="button" href="{{'/get-started/quickstart-py/'|prepend:site.baseurl}}">Python 快速入门</a>
  </div>
</div>
