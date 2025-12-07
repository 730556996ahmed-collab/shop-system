<!doctype html>
<html lang="ar" dir="rtl">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>نظام التسوق والإدارة</title>
<style>
:root{--bg:#0f1724;--card:#0b1220;--accent:#06b6d4;--muted:#9aa4b2;}
*{box-sizing:border-box;margin:0;padding:0;}
body{font-family:Tahoma,Arial,sans-serif;background:var(--bg);color:#e6eef5;min-height:100vh;display:flex;flex-direction:column;}
header{padding:16px 20px;border-bottom:1px solid rgba(255,255,255,0.04);display:flex;align-items:center;justify-content:space-between;}
.brand{display:flex;align-items:center;gap:12px;}
.logo{width:50px;height:50px;border-radius:12px;background:linear-gradient(135deg,var(--accent),#7c3aed);display:flex;align-items:center;justify-content:center;font-weight:700;color:#042028;}
h1{font-size:20px;margin:0;}
p.lead{color:var(--muted);}
main{flex:1;padding:20px;max-width:1100px;margin:0 auto;width:100%;}
.centers{display:grid;grid-template-columns:repeat(auto-fit,minmax(200px,1fr));gap:12px;margin-top:20px;}
.card{background:var(--card);padding:12px;border-radius:12px;display:flex;flex-direction:column;justify-content:space-between;border:1px solid rgba(255,255,255,0.03);}
.card h3{margin-bottom:6px;}
.card p{color:var(--muted);font-size:14px;}
.button{padding:6px 10px;border-radius:6px;border:none;background:linear-gradient(90deg,var(--accent),#7c3aed);color:#042028;cursor:pointer;margin-top:6px;}
.overlay{position:fixed;inset:0;background:rgba(2,6,23,0.9);display:none;align-items:center;justify-content:center;padding:20px;z-index:100;}
.overlay.active{display:flex;}
.modal{background:#02111a;padding:18px;border-radius:12px;max-width:900px;width:100%;box-shadow:0 10px 30px rgba(2,6,23,0.6);}
.grid-cols{display:grid;grid-template-columns:1fr 360px;gap:16px;}
.product-list{max-height:60vh;overflow:auto;padding-right:8px;}
.section{background:linear-gradient(180deg,rgba(255,255,255,0.02),transparent);padding:10px;border-radius:8px;margin-bottom:10px;}
.section h5{margin-bottom:6px;}
.item{display:flex;align-items:center;justify-content:space-between;padding:6px 8px;border-radius:6px;background:rgba(255,255,255,0.01);margin-bottom:6px;}
.item img{width:50px;height:50px;border-radius:6px;margin-left:6px;}
.cart{background:#061823;padding:10px;border-radius:8px;}
.cart h5{margin-bottom:6px;}
.field{display:flex;flex-direction:column;margin-bottom:8px;}
.field label{font-size:13px;color:var(--muted);margin-bottom:4px;}
.field input,.field textarea{padding:8px;border-radius:6px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:inherit;}
.send{width:100%;padding:8px;border-radius:6px;border:none;background:linear-gradient(90deg,var(--accent),#7c3aed);color:#042028;cursor:pointer;margin-top:6px;}
.admin-panel{display:none;margin-top:20px;}
.admin-panel h2{margin-bottom:10px;}
.admin-panel .admin-section{margin-bottom:14px;padding:10px;background:var(--card);border-radius:10px;}
.admin-panel table{width:100%;border-collapse:collapse;margin-top:6px;}
.admin-panel table, .admin-panel th, .admin-panel td{border:1px solid #444;}
.admin-panel th, .admin-panel td{padding:6px;text-align:center;}
@media(max-width:720px){.grid-cols{grid-template-columns:1fr}}
</style>
</head>
<body>
<header>
  <div class="brand">
    <div class="logo">أع</div>
    <div>
      <h1>نظام التسوق والإدارة</h1>
      <p class="lead">الواجهة المزدوجة</p>
    </div>
  </div>
  <button class="button" id="adminLoginBtn">دخول الإدارة</button>
</header>

<main>
  <div id="centersContainer" class="centers"></div>
</main>

<!-- واجهة الزبون -->
<div class="overlay" id="shopOverlay">
  <div class="modal">
    <header style="display:flex;justify-content:space-between;align-items:center;">
      <h4 id="shopTitle">المركز</h4>
      <button class="send" id="closeShop">إغلاق</button>
    </header>
    <div class="grid-cols">
      <div class="product-list" id="sectionsList"></div>
      <aside>
        <div class="cart">
          <h5>قائمة المشتريات</h5>
          <div id="cartItems"></div>
          <div class="field"><label>الاسم</label><input id="custName" placeholder="اسمك"></div>
          <div class="field"><label>رقم الهاتف</label><input id="custPhone" placeholder="رقم الهاتف"></div>
          <div class="field"><label>ملاحظات</label><textarea id="custNotes" rows="3" placeholder="ملاحظات أو عنوان"></textarea></div>
          <button class="send" id="sendOrder">إرسال الطلب</button>
        </div>
      </aside>
    </div>
  </div>
</div>

<!-- واجهة تسجيل الدخول للإدارة -->
<div class="overlay" id="adminLoginOverlay">
  <div class="modal">
    <h3>تسجيل دخول الإدارة</h3>
    <div class="field">
      <label>كلمة المرور</label>
      <input type="password" id="adminPassword">
    </div>
    <button class="send" id="adminSubmitBtn">دخول</button>
    <button class="send" id="adminCancelBtn">إلغاء</button>
  </div>
</div>

<!-- لوحة الإدارة -->
<div class="admin-panel" id="adminPanel">
  <h2>لوحة الإدارة</h2>
  <button class="button" id="addCenterBtn">أضف مركز جديد</button>
  <div id="adminCentersList" style="margin-top:12px"></div>
</div>

<script>
const ADMIN_PASSWORD='admin123';
let centers=JSON.parse(localStorage.getItem('centers_shop')||'[]');

const centersContainer=document.getElementById('centersContainer');
function renderCenters(){
  centersContainer.innerHTML='';
  centers.forEach(c=>{
    const card=document.createElement('div'); card.className='card';
    const h=document.createElement('h3'); h.textContent=c.title; card.appendChild(h);
    const p=document.createElement('p'); p.textContent=c.description||'أقسام متنوعة'; card.appendChild(p);
    const btn=document.createElement('button'); btn.className='button'; btn.textContent='ادخل للتسوق';
    btn.addEventListener('click',()=>openShop(c.id)); card.appendChild(btn);
    centersContainer.appendChild(card);
  });
}
renderCenters();

// ================== SHOP ==================
const shopOverlay=document.getElementById('shopOverlay');
const sectionsList=document.getElementById('sectionsList');
const shopTitle=document.getElementById('shopTitle');
const closeShop=document.getElementById('closeShop');
let currentCenter=null;
let cart=[];

function openShop(centerId){
  currentCenter=centers.find(c=>c.id===centerId);
  if(!currentCenter)return;
  shopTitle.textContent=currentCenter.title;
  sectionsList.innerHTML='';
  (currentCenter.sections||[]).forEach(sec=>{
    const sdiv=document.createElement('div'); sdiv.className='section';
    const h=document.createElement('h5'); h.textContent=sec.title; sdiv.appendChild(h);
    (sec.items||[]).forEach(it=>{
      const itDiv=document.createElement('div'); itDiv.className='item';
      const txt=document.createElement('div'); txt.textContent=`${it.name} — ${it.price} ريال`;
      if(it.img){ const img=document.createElement('img'); img.src=it.img; itDiv.appendChild(img); }
      const addBtn=document.createElement('button'); addBtn.className='send'; addBtn.textContent='أضف';
      addBtn.addEventListener('click',()=>{ cart.push({...it, section:sec.title, center:currentCenter.title}); updateCartUI(); });
      itDiv.appendChild(txt); itDiv.appendChild(addBtn); sdiv.appendChild(itDiv);
    });
    sectionsList.appendChild(sdiv);
  });
  cart=[]; updateCartUI();
  shopOverlay.classList.add('active');
}
closeShop.addEventListener('click',()=>{ shopOverlay.classList.remove('active'); cart=[]; updateCartUI(); });

function updateCartUI(){
  const cartDiv=document.getElementById('cartItems'); cartDiv.innerHTML='';
  if(cart.length===0){ cartDiv.innerHTML='<div class="field small">لا توجد مشتريات بعد.</div>'; return; }
  cart.forEach((it,i)=>{
    const d=document.createElement('div'); d.style.display='flex'; d.style.justifyContent='space-between';
    const txt=document.createElement('div'); txt.textContent=`${i+1}. ${it.name} — ${it.price} ريال`;
    const rem=document.createElement('button'); rem.className='send'; rem.textContent='×'; rem.addEventListener('click',()=>{ cart.splice(i,1); updateCartUI(); });
    d.appendChild(txt); d.appendChild(rem); cartDiv.appendChild(d);
  });
}

document.getElementById('sendOrder').addEventListener('click',()=>{
  const name=document.getElementById('custName').value.trim();
  const phone=document.getElementById('custPhone').value.trim();
  const notes=document.getElementById('custNotes').value.trim();
  if(!name||!phone||cart.length===0){ alert('الرجاء إدخال الاسم، الهاتف، واختيار منتج واحد على الأقل'); return; }
  // حفظ الطلب في المركز والقسم
  cart.forEach(it=>{
    const center=centers.find(c=>c.title===it.center);
    if(!center.orders) center.orders=[];
    center.orders.push({customer:{name,phone,notes},item:it});
  });
  localStorage.setItem('centers_shop',JSON.stringify(centers));
  alert('تم إرسال الطلب للوحة الإدارة!');
  cart=[]; updateCartUI();
});

// ================== ADMIN ==================
const adminLoginOverlay=document.getElementById('adminLoginOverlay');
document.getElementById('adminLoginBtn').addEventListener('click',()=>{ adminLoginOverlay.classList.add('active'); });
document.getElementById('adminCancelBtn').addEventListener('click',()=>{ adminLoginOverlay.classList.remove('active'); });

document.getElementById('adminSubmitBtn').addEventListener('click',()=>{
  const pass=document.getElementById('adminPassword').value;
  if(pass!==ADMIN_PASSWORD){ alert('كلمة المرور خاطئة'); return; }
  adminLoginOverlay.classList.remove('active');
  document.getElementById('adminPanel').style.display='block';
  renderAdminCenters();
});

const adminCentersList=document.getElementById('adminCentersList');

function renderAdminCenters(){
  adminCentersList.innerHTML='';
  centers.forEach((c,ci)=>{
    const cDiv=document.createElement('div'); cDiv.className='admin-section';
    const titleInput=document.createElement('input'); titleInput.value=c.title; titleInput.placeholder='عنوان المركز';
    const descInput=document.createElement('input'); descInput.value=c.description||''; descInput.placeholder='وصف المركز';
    const saveBtn=document.createElement('button'); saveBtn.textContent='حفظ'; saveBtn.className='button';
    saveBtn.addEventListener('click',()=>{
      c.title=titleInput.value; c.description=descInput.value; saveAndRender();
    });
    const delBtn=document.createElement('button'); delBtn.textContent='حذف'; delBtn.className='button';
    delBtn.addEventListener('click',()=>{ if(confirm('حذف المركز؟')){ centers.splice(ci,1); saveAndRender(); } });
    const addSecBtn=document.createElement('button'); addSecBtn.textContent='أضف قسم'; addSecBtn.className='button';
    addSecBtn.addEventListener('click',()=>{
      const secTitle=prompt('عنوان القسم'); if(!secTitle)return;
      c.sections=c.sections||[]; c.sections.push({id:'s'+Date.now(),title:secTitle,items:[]}); saveAndRender();
    });
    const secDiv=document.createElement('div');
    (c.sections||[]).forEach((s,si)=>{
      const sDiv=document.createElement('div'); sDiv.style.border='1px solid #444'; sDiv.style.margin='6px 0'; sDiv.style.padding='6px';
      const sInput=document.createElement('input'); sInput.value=s.title; sInput.placeholder='عنوان القسم';
      sInput.addEventListener('change',()=>{ s.title=sInput.value; saveAndRender(); });
      const delSBtn=document.createElement('button'); delSBtn.textContent='حذف قسم'; delSBtn.className='button';
      delSBtn.addEventListener('click',()=>{ c.sections.splice(si,1); saveAndRender(); });
      const addItemBtn=document.createElement('button'); addItemBtn.textContent='أضف منتج'; addItemBtn.className='button';
      addItemBtn.addEventListener('click',()=>{
        const name=prompt('اسم المنتج'); if(!name)return;
        const price=prompt('السعر'); if(!price)return;
        const img=prompt('رابط الصورة (اختياري)')||'';
        s.items=s.items||[]; s.items.push({name,price,img}); saveAndRender();
      });
      // جدول المنتجات
      let table=document.createElement('table');
      let tr=document.createElement('tr'); tr.innerHTML='<th>المنتج</th><th>السعر</th><th>الصورة</th>'; table.appendChild(tr);
      (s.items||[]).forEach(it=>{
        const tr=document.createElement('tr');
        tr.innerHTML=`<td>${it.name}</td><td>${it.price}</td><td>${it.img?'<img src="'+it.img+'" width="50">':''}</td>`;
        table.appendChild(tr);
      });
      sDiv.appendChild(sInput); sDiv.appendChild(delSBtn); sDiv.appendChild(addItemBtn); sDiv.appendChild(table);
      secDiv.appendChild(sDiv);
    });
    // جدول الطلبات
    if(c.orders && c.orders.length>0){
      const ordersDiv=document.createElement('div'); ordersDiv.style.marginTop='10px';
      ordersDiv.innerHTML='<h4>الطلبات</h4>';
      c.orders.forEach(o=>{
        const t=document.createElement('table');
        t.innerHTML='<tr><th>الاسم</th><th>الهاتف</th><th>ملاحظات</th><th>المنتج</th><th>السعر</th><th>قسم</th></tr>';
        const tr=document.createElement('tr');
        tr.innerHTML=`<td>${o.customer.name}</td><td>${o.customer.phone}</td><td>${o.customer.notes}</td><td>${o.item.name}</td><td>${o.item.price}</td><td>${o.item.section}</td>`;
        t.appendChild(tr); ordersDiv.appendChild(t);
      });
      cDiv.appendChild(ordersDiv);
    }
    cDiv.appendChild(titleInput); cDiv.appendChild(descInput); cDiv.appendChild(saveBtn); cDiv.appendChild(delBtn); cDiv.appendChild(addSecBtn); cDiv.appendChild(secDiv);
    adminCentersList.appendChild(cDiv);
  });
}

document.getElementById('addCenterBtn').addEventListener('click',()=>{
  centers.push({id:'c'+Date.now(),title:'مركز جديد',description:'وصف',sections:[],orders:[]}); saveAndRender();
});

function saveAndRender(){
  localStorage.setItem('centers_shop',JSON.stringify(centers));
  renderCenters();
  renderAdminCenters();
}
</script>

</body>
</html>
