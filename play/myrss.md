---
layout: page
type: play
title: RSS源集合
---
  
  
<script type="text/javascript" src="{{ site.js | relative_url }}/src/myrss.js"></script>

  <div id="comments" style="margin-top: 50px;"></div>
  <!--Leancloud 操作库:-->
  <script src="//cdn1.lncld.net/static/js/3.0.4/av-min.js"></script>
  <!--Valine 的核心代码库:-->
  <script src='//unpkg.com/valine/dist/Valine.min.js'></script>
  <script>
     new Valine({
        av: AV,
        el: '#comments',
        app_id: 'e1OuTd58aBj3h9ptV4oIaNBY-9Nh9j0Va',
        app_key: 'CqYVue1Ivtz4TJnBVjUvY9NY',
        path: '',
        placeholder: '可以给我留言免费帮忙定制！',
        notify: 'true',
        verify: 'true',
    })
  </script> 