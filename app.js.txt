const SUPABASE_URL = 'https://yjclkfzidxoawbpkrtus.supabase.co';
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InlqY2xrZnppZHhvYXdicGtydHVzIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODA3MTIzOTEsImV4cCI6MjA5NjI4ODM5MX0.eWKqdNbLxSJkQK0KKSp8p1OX3Ybsz1mmAv0DYiER9oo';

const Toast={_el:null,_timer:null,show(msg,t=2000){if(!this._el){this._el=document.createElement('div');this._el.style.cssText='position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);background:rgba(0,0,0,0.75);color:white;padding:12px 24px;border-radius:8px;font-size:14px;z-index:9999;pointer-events:none;opacity:0;transition:opacity 0.3s;';document.body.appendChild(this._el)}clearTimeout(this._timer);this._el.textContent=msg;this._el.style.opacity='1';this._timer=setTimeout(()=>{this._el.style.opacity='0'},t)}};
function showConfirm(msg){return new Promise(r=>{const m=document.createElement('div');m.style.cssText='position:fixed;inset:0;background:rgba(0,0,0,0.5);z-index:500;display:flex;align-items:center;justify-content:center;';m.innerHTML='<div style="background:white;border-radius:16px;padding:24px;width:80%;max-width:300px;text-align:center;"><div style="font-size:40px;">🤔</div><div style="font-size:17px;color:#333;margin:12px 0 20px;">'+msg+'</div><div style="display:flex;gap:12px;"><button id="no" style="flex:1;padding:12px;border-radius:10px;border:none;background:#f5f5f5;color:#666;">取消</button><button id="yes" style="flex:1;padding:12px;border-radius:10px;border:none;background:#DEAAD3;color:white;">确认</button></div></div>';document.body.appendChild(m);m.querySelector('#no').onclick=()=>{m.remove();r(false)};m.querySelector('#yes').onclick=()=>{m.remove();r(true)}})}

async function api(p,o={}){const h={'apikey':SUPABASE_KEY,'Authorization':'Bearer '+SUPABASE_KEY};if(o.body){h['Content-Type']='application/json';o.body=JSON.stringify(o.body)}const r=await fetch(SUPABASE_URL+'/rest/v1/'+p,{...o,headers:h});if(r.status===204)return null;if(!r.ok){const e=await r.text();console.error(e);throw new Error(e)}return r.json()}

async function uploadImage(file){const safeName=Date.now()+'_'+file.name.replace(/[^a-zA-Z0-9._-]/g,'_');const res=await fetch(SUPABASE_URL+'/storage/v1/object/food-images/'+safeName,{method:'POST',headers:{'Authorization':'Bearer '+SUPABASE_KEY},body:file});if(!res.ok)return'';return SUPABASE_URL+'/storage/v1/object/public/food-images/'+safeName}

const CATS=['注意‼️','小狗券','小狗心情','小狗口味','小狗主厨','小狗美食','小狗甜品','小狗运动','小狗饮料'];
let page='home',catIdx=0,cart=[],orders=[],foods=[];

async function init(){await loadFoods();loadCart();await loadOrders();await loadFavs();renderHome();bindTabs()}
async function loadFoods(){try{const d=await api('foods?select=*&order=id');const g=[[],[],[],[],[],[],[],[],[]];d.forEach(f=>{const c=f.category_index||0;if(!g[c])g[c]=[];g[c].push({id:f.id,name:f.name,emoji:f.emoji,price:f.price,catIdx:f.category_index,img:f.image_url,fav:false})});foods=g}catch(e){Toast.show('加载失败')}}
function loadCart(){cart=JSON.parse(localStorage.getItem('cart')||'[]')}
async function loadOrders(){try{const d=await api('orders?select=*&order=created_at.desc');orders=d.map(o=>({id:o.id,time:new Date(o.created_at).toLocaleString('zh-CN'),items:o.items,total:o.total_price,status:o.status}))}catch(e){orders=[]}}
async function loadFavs(){try{const d=await api('favorites?select=food_id');const ids=d.map(f=>f.food_id);foods.forEach(c=>c.forEach(f=>f.fav=ids.includes(f.id)))}catch(e){}}
function saveCart(){localStorage.setItem('cart',JSON.stringify(cart))}
function renderPage(c){document.getElementById('page-container').innerHTML=c}

function renderHome(){
  const cf=foods[catIdx]||[];
  const tc=cart.reduce((s,i)=>s+i.count,0),tp=cart.reduce((s,i)=>s+(i.price||0)*i.count,0).toFixed(2);
  let h='<div class="header-bg"><div class="header-top"><div class="shop-name">今天吃什么</div></div><div class="header-info"><div class="shop-logo"><div class="logo-box">🍳</div></div><div class="shop-info-text"><div class="shop-title">今天吃什么</div><div class="shop-count">共'+foods.flat().length+'个菜谱</div></div></div></div>';
  h+='<div class="main-content"><div class="category-sidebar">';
  CATS.forEach((c,i)=>{h+='<div class="category-item'+(i===catIdx?' active':'')+'" onclick="switchCat('+i+')">'+(i===catIdx?'<div class="category-indicator"></div>':'')+'<span class="category-text'+(i===catIdx?' active-text':'')+'">'+c+'</span></div>'});
  h+='</div><div class="food-scroll"><div class="category-title"><span class="title-text">'+CATS[catIdx]+'</span></div><div class="food-list">';
  cf.forEach(f=>{h+='<div class="food-card"><div class="food-img-box">'+(f.img?'<img class="food-img-real" src="'+f.img+'"/>':'<div class="food-img-placeholder">'+(f.emoji||'🍽️')+'</div>')+'</div><div class="food-info"><div class="food-name">'+f.name+'</div><div class="food-stars">⭐⭐⭐⭐⭐</div><div class="food-price-row"><span class="food-price">¥'+Number(f.price).toFixed(2)+'</span><button class="fav-btn" onclick="toggleFav('+f.id+')"><span class="fav-icon">'+(f.fav?'❤️':'🤍')+'</span></button></div></div><div class="food-add"><button class="add-btn" onclick="addCart('+f.id+')">+</button></div></div>'});
  h+='</div></div></div><div class="bottom-float-bar"><div class="float-cart" onclick="goCart()"><span class="cart-icon">🛒</span>'+(tc>0?'<span class="cart-badge">'+(tc>99?'99+':tc)+'</span>':'')+(tp>0?'<span class="cart-price">¥'+tp+'</span>':'')+'</div><div class="float-btns"><button class="btn-order'+(tc>0?' active':'')+'" onclick="submitOrder()">下单</button></div></div>';
  renderPage(h);
}

function switchCat(i){catIdx=i;renderHome()}
function addCart(id){const f=foods.flat().find(x=>x.id===id);if(!f)return;const e=cart.find(x=>x.id===id);e?e.count++:cart.push({...f,count:1});saveCart();renderHome();Toast.show('已加入')}
async function toggleFav(id){const f=foods.flat().find(x=>x.id===id);if(!f)return;if(f.fav){await api('favorites?food_id=eq.'+id,{method:'DELETE'});Toast.show('已取消')}else{await api('favorites',{method:'POST',body:{food_id:id}});Toast.show('已收藏')}f.fav=!f.fav;renderHome()}
function goCart(){renderCart()}
function submitOrder(){if(!cart.length){Toast.show('请先选菜品');return}renderCart()}

function renderCart(){
  const tp=cart.reduce((s,i)=>s+(i.price||0)*i.count,0).toFixed(2);
  let h='<div class="page-title">🛒 购物车</div>';
  if(!cart.length)h+='<div class="empty-state"><div class="empty-icon">🍽️</div><div class="empty-text">购物车是空的</div><button class="go-menu-btn" onclick="switchTab(\'home\')">去点餐</button></div>';
  else{h+='<div class="cart-list">';cart.forEach(i=>{h+='<div class="cart-item"><div class="cart-img-box">'+(i.img?'<img class="cart-img-real" src="'+i.img+'"/>':'<div class="cart-emoji">'+(i.emoji||'🍽️')+'</div>')+'</div><div class="cart-info"><div class="cart-name">'+i.name+'</div><div class="cart-price">¥'+Number(i.price).toFixed(2)+' / 份</div><div class="cart-bottom"><div class="cart-actions"><button class="btn-minus" onclick="updCart('+i.id+',-1)">−</button><span class="cart-count">'+i.count+'</span><button class="btn-plus" onclick="updCart('+i.id+',1)">+</button></div><span class="item-total">¥'+(i.price*i.count).toFixed(2)+'</span></div></div></div>'});h+='</div><div class="bottom-bar"><div class="total-info"><span class="total-label">合计：</span><span class="total-price">¥'+tp+'</span></div><button class="submit-btn" onclick="confirmOrder()">提交订单</button></div>'}
  renderPage(h);
}

function updCart(id,d){const i=cart.find(x=>x.id===id);if(!i)return;i.count=Math.max(0,i.count+d);if(!i.count)cart=cart.filter(x=>x.id!==id);saveCart();renderCart()}

async function confirmOrder(){
  if(!cart.length){Toast.show('购物车为空');return}
  if(!await showConfirm('共 '+cart.reduce((s,i)=>s+i.count,0)+' 件商品，确认下单？'))return;
  try{
    const body={items:cart.map(c=>({id:c.id,name:c.name,emoji:c.emoji,count:c.count,price:c.price})),total_price:Number(cart.reduce((s,i)=>s+(i.price||0)*i.count,0)),status:'待接单'};
    const res=await fetch(SUPABASE_URL+'/rest/v1/orders',{method:'POST',headers:{'apikey':SUPABASE_KEY,'Authorization':'Bearer '+SUPABASE_KEY,'Content-Type':'application/json','Prefer':'return=minimal'},body:JSON.stringify(body)});
    if(!res.ok){const e=await res.text();console.error(e);Toast.show('下单失败');return}
    cart=[];saveCart();Toast.show('下单成功！等待商家接单 🎉');setTimeout(()=>switchTab('order'),1000);
  }catch(e){console.error(e);Toast.show('下单失败')}
}

async function renderOrder(){await loadOrders();let h='<div class="page-title">📋 我的订单</div>';if(!orders.length)h+='<div class="empty-state"><div class="empty-icon">📭</div><div class="empty-text">暂无订单</div><button class="go-menu-btn" onclick="switchTab(\'home\')">去点餐</button></div>';else{h+='<div class="order-list">';orders.forEach((o,i)=>{const statusColor=o.status==='待接单'?'#FF9800':o.status==='制作中'?'#2196F3':'#4CAF50';h+='<div class="order-card"><div class="order-header"><span>订单 #'+String(i+1).padStart(3,'0')+'</span><span class="order-status" style="color:'+statusColor+'">'+o.status+'</span></div><div class="order-time">'+o.time+'</div><div class="order-items">';o.items.forEach(it=>{h+='<div class="order-item"><span>'+it.emoji+'</span><span>'+it.name+'</span><span>x'+it.count+'</span></div>'});h+='</div><div class="order-footer">合计：<span class="total-price">¥'+Number(o.total).toFixed(2)+'</span></div></div>'});h+='</div>'}renderPage(h)}

function renderMine(){const fc=foods.flat().filter(f=>f.fav).length;renderPage('<div class="user-header"><div class="user-avatar">🐶</div><div class="user-name">今天吃什么</div></div><div class="menu-list"><div class="menu-item" onclick="switchTab(\'order\')"><span class="menu-icon">📋</span><span class="menu-text">我的订单</span><span class="menu-arrow">›</span></div><div class="menu-item" onclick="goFavs()"><span class="menu-icon">⭐</span><span class="menu-text">我的收藏</span>'+(fc>0?'<span class="menu-badge">'+fc+'</span>':'')+'<span class="menu-arrow">›</span></div></div>')}
function goFavs(){const ff=foods.flat().filter(f=>f.fav);let h='<div class="page-title">⭐ 我的收藏</div>';if(!ff.length)h+='<div class="empty-state"><div class="empty-icon">🤍</div><div class="empty-text">还没有收藏</div><button class="go-menu-btn" onclick="switchTab(\'home\')">去点餐</button></div>';else{h+='<div class="fav-list">';ff.forEach(f=>{h+='<div class="fav-item"><div class="fav-img-box">'+(f.img?'<img src="'+f.img+'"/>':'<div class="fav-emoji">'+f.emoji+'</div>')+'</div><div class="fav-info"><div class="fav-name">'+f.name+'</div><div class="fav-price">¥'+Number(f.price).toFixed(2)+'</div></div><button class="fav-btn" onclick="toggleFav('+f.id+');goFavs()">❤️</button><button class="add-btn" onclick="addCart('+f.id+')">+</button></div>'});h+='</div>'}renderPage(h)}

function bindTabs(){document.querySelectorAll('.tab-item').forEach(t=>{t.addEventListener('click',function(){switchTab(this.dataset.page)})})}
function switchTab(p){page=p;document.querySelectorAll('.tab-item').forEach(t=>t.classList.toggle('active',t.dataset.page===p));if(p==='home')renderHome();else if(p==='order')renderOrder();else if(p==='mine')renderMine()}
function showToast(msg){Toast.show(msg)}
document.addEventListener('DOMContentLoaded',init);