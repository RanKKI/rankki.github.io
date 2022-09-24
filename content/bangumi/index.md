---
title: "Bangumi"
---

下面是一些我看过或者想看的动漫，数据来源 [api.bgm.tv](https://bangumi.github.io/api/)。

出于各种原因考虑，最多只显示 100 条。全部内容可以在 [bgm.tv/anime/list/725320](https://bgm.tv/anime/list/725320)  看到。

<div id="bangumi_container">
<div class="loading_container">
<div class="lds-ellipsis"><div></div><div></div><div></div><div></div></div>
</div>
</div>
<script type='text/javascript'>
  const groupBy = function (xs, key) {
    return xs.reduce(function (rv, x) {
      (rv[x[key]] = rv[x[key]] || []).push(x);
      return rv;
    }, {});
  };
  const collection_types = {
    1: "想看",
    2: "看过",
    3: "在看",
    4: "搁置",
    5: "抛弃",
  }
  const sort_key = [3, 1, 2, 4, 5]
  const insert = (item) => {
    $(`#bangumi_container${item.type}`).append(`<div class="column" style="margin-left: 10px;"><a href="https://bgm.tv/subject/${item.subject.id}"><img src="${item.subject.images.medium}" /></a></div>`)
  }
  function func() {
    let api = "https://api.bgm.tv"
    // api = "http://localhost:8081"
    $.getJSON(api + "/v0/users/725320/collections?subject_type=2&limit=100&offset=0")
      .then(data => {
        $("#bangumi_container").empty()
        let ret = groupBy(data.data ?? [], "type")
        Object.keys(ret).sort((a, b) => {
          return sort_key.indexOf(parseInt(a)) - sort_key.indexOf(parseInt(b))
        }).forEach(key => {
          let first = ret[key][0]
          if (first) {
            $("#bangumi_container").append(`<h1>${collection_types[first.type]}</h1><div id="bangumi_container${first.type}" style="display:flex; flex-flow:row wrap; justify-content:left; align-items:center;"></div>`)
          }
          ret[key].forEach(item => {
            insert(item)
          })
        })
      })
      .fail((obj) => {
        $("#bangumi_container").append(`<p>加载失败..</p>`)
      })
  }
  function callback() {
    if (window.$) {
      func();
    } else {
      // wait 50 milliseconds and try again.
      window.setTimeout(callback, 50);
    }
  }
  // callback();
</script>