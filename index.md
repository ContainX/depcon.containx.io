---
layout: project
project: depcon
---

<div class="feature intro">
  <div class="container">
    <div class="row">
      <div class="col-sm-12">
        <p>
Depcon makes managing clusters that run docker containers a breeze. It offers the ability to define environments such as test, pre-prod, production against clusters supporting Apache Mesos & Marathon (current release), Kubernetes and Amazon ECS.  Depcon also provides a zero downtime deployments for Marathon known as Blue-Green.
        </p>
      </div>
    </div>
  </div>
</div>

<!-- Blue/Green -->
<div class="feature gray-background">
  <div class="container">
    <div class="row">
      <div class="col-sm-6">
        <h2>Blue-Green</h2>
        <p>Still using maintenance pages but would like to do hot deployments?  Depcon supports Blue-Green deployments against Marathon-LB.  Future releases will support NGINX and other proxies. </p>
      </div>
      <div class="col-sm-5 text-right">
        <img class="svgcolor" src="/assets/img/icons/bluegreen.svg">
      </div>
    </div>
  </div>
</div>

<!-- Automates -->
<div class="feature white-background">
  <div class="container">
    <div class="row">      
      <div class="col-sm-5 col-sm-offset-1">
        <img class="svg" src="/assets/img/icons/automates.svg">
      </div>
      <div class="col-sm-6 text-right">
        <h2>Automates</h2>
        <p>Depcon is extremely easy to integrate into your existing CI/Delivery pipeline. It provides an easy way to deploy containers across all your clusters.  Depcon supports blocking on health status which is key to provide fast feedback to engineering.</p>
      </div>
    </div>
  </div>
</div>

<!--Scalable -->
<div class="feature gray-background">
  <div class="container">
    <div class="row">
      <div class="col-sm-6">
        <h2>Simplify</h2>
        <p>Depcon supports templates and offers full variable substitution. It also has name defined environment allowing you to deploy across different environments without the need of remember URL's and authentication.</p>
      </div>
      <div class="col-sm-5 text-right">
        <img class="svg" src="/assets/img/icons/simplify.svg">
      </div>
    </div>
  </div>
</div>

<!-- Embeddable -->
<!-- <div class="feature gray-background">
  <div class="container">
    <div class="row">
<div class="col-sm-6" markdown="1">
```java
containxReplica replica = containxReplica.builder(address, members)
  .withTransport(new NettyTransport())
  .withStorage(new Storage(StorageLevel.DISK))
  .build()
  .open()
  .get();
```
</div>
      <div class="col-sm-6 text-right">
        <h2>Embeddable</h2>
        <p>containx supports fully embeddable replicas that live in-process, eliminating the need to manage external coordination services.</p>
      </div>
    </div>
  </div>
</div> -->

<!--Learn more -->
<div class="feature get-started">
  <div class="container">
    <div class="row">
      <div class="col-sm-12 text-center">
        <h2>Ready to learn more?</h2>
        <p>
          <a href="/docs/getting-started" class="btn btn-success btn-lg doc-btn">Get Started</a>
        </p>
      </div>
    </div>
  </div>
</div>

<script type='text/javascript'>
// Format tabs
$(function(){
  var $container = $('#sync-tabs');

  updateTabs($container);
  $(window).resize(function(){
    updateTabs($container);
  })

  function updateTabs($tabsContainer){
      var $containerWidth = $tabsContainer.width();
      var tabWidths = [];
      var $tabs = $tabsContainer.find('li');
      $tabs.each(function(index, tab){
        tabWidths.push($(tab).width());
      });

      var formattedTabs = [];
      var maxWidth = $containerWidth;
      var maxWidthSet = false;
      var rowWidth = 0;
      for(var i = tabWidths.length - 1; i >= 0; i--){
          var tabWidth = tabWidths[i];
          if(rowWidth + tabWidth > maxWidth){
            if(!maxWidthSet){
              maxWidth = rowWidth;
              maxWidthSet = true;
            }
            rowWidth = tabWidth;
            formattedTabs.unshift($('<div class="spacer"></div>'));
          }else{
            rowWidth += tabWidth;
          }
          formattedTabs.unshift($tabs.get(i));
      }

      var $tempContainer = $('<div></div>');
      formattedTabs.forEach(function(tab, index){
        $tempContainer.append(tab);
      });
      $tabsContainer.html($tempContainer.html());
  }
});
</script>
