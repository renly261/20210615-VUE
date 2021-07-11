<template lang="pug">
#app
  b-navbar(toggable='lg' type='dark' variant='info')
    b-container
      b-navbar-brand(to='/') 番茄鐘
      b-navbar-toggle(target='navbar-collapse')
      b-collapse#navbar-collapse(is-nav)
        b-navbar-nav.ml-auto
          b-nav-item(to='/list') 清單
          b-nav-item(to='/settings') 設定
  //- 有倒數之類的東西要用 keep-alive 把 router-view 包住, 防止換頁時東西被清掉導致倒數錯誤
  keep-alive
    //- 用 router-view 換頁的時候東西資料會被清掉
    router-view
</template>
