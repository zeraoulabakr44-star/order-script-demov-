<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>طلب المنتج</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
:root{
  --main:#25D366;
  --accent:#ff7a00;
  --bg:#f2f4f8;
}
body{
  font-family:Tajawal,Tahoma,Arial;
  background:var(--bg);
  margin:0;
}
.box{
  max-width:420px;
  background:#fff;
  margin:20px auto;
  padding:18px;
  border-radius:18px;
  box-shadow:0 15px 40px rgba(0,0,0,.12);
}
.product-img{
  width:100%;
  aspect-ratio:1/1;
  object-fit:cover;
  border-radius:16px;
  margin-bottom:12px;
}
.title{
  text-align:center;
  font-size:22px;
  font-weight:800;
}
.price-main{
  text-align:center;
  font-size:24px;
  color:var(--accent);
  font-weight:900;
  margin-bottom:15px;
}
label{
  display:block;
  margin-top:12px;
  font-weight:700;
  font-size:14px;
}
input,select{
  width:100%;
  padding:13px;
  margin-top:6px;
  border-radius:12px;
  border:1px solid #ddd;
  font-size:15px;
  background:#fafafa;
}
.qty-box{
  display:flex;
  gap:10px;
  margin-top:6px;
}
.qty-box button{
  width:44px;
  height:44px;
  border-radius:12px;
  border:none;
  background:#eee;
  font-size:22px;
  font-weight:bold;
}
.summary{
  background:#f9fafb;
  padding:14px;
  border-radius:14px;
  margin-top:16px;
  border:1px dashed #ddd;
}
.summary div{
  display:flex;
  justify-content:space-between;
  margin-bottom:6px;
}
.summary .total{
  font-size:17px;
  font-weight:900;
  color:var(--accent);
}
#orderBtn{
  width:100%;
  margin-top:20px;
  padding:16px;
  background:linear-gradient(135deg,var(--main),#1ebe5d);
  color:#fff;
  border:none;
  border-radius:16px;
  font-size:18px;
  font-weight:900;
  cursor:pointer;
}
.modal{
  position:fixed;
  inset:0;
  background:rgba(0,0,0,.6);
  display:none;
  align-items:center;
  justify-content:center;
}
.modal-box{
  background:#fff;
  width:90%;
  max-width:380px;
  padding:20px;
  border-radius:18px;
}
.modal-actions{
  display:flex;
  gap:10px;
  margin-top:15px;
}
.modal-actions button{
  flex:1;
  padding:13px;
  border:none;
  border-radius:12px;
  font-size:16px;
  font-weight:700;
}
.confirm{background:var(--main);color:#fff}
.cancel{background:#eee}
</style>
</head>
<body>

<script>
// ⚙️ إعدادات المنتج
var SETTINGS = {
  whatsapp: "213555000000", // رقم واتساب للتواصل
  deliveryTypes: {"المنزل":0,"المكتب":-100},
  deliveryPrices: {
    "الجزائر":400,
    "وهران":600,
    "عنابة":700,
    "قسنطينة":700
  }
};

// قائمة المنتجات
var PRODUCTS = [
  {name:"ساعة رجالية – موديل A", price:3000, image:"images/model-a.jpg"},
  {name:"ساعة رجالية – موديل B", price:3500, image:"images/model-b.jpg"}
];
</script>

<div class="box">
  <img id="productImg" class="product-img">
  <div class="title">اختر الموديل</div>
  <select id="productSelect"></select>
  <div class="price-main" id="productPrice"></div>

  <label>الاسم الكامل</label>
  <input id="name">
  <label>رقم الهاتف</label>
  <input id="phone">

  <label>الكمية</label>
  <div class="qty-box">
    <button id="minus">−</button>
    <input id="qty" type="number" value="1" min="1">
    <button id="plus">+</button>
  </div>

  <label>الولاية</label>
  <select id="wilaya"><option value="">اختر الولاية</option></select>

  <label>نوع التوصيل</label>
  <select id="deliveryType"><option value="">اختر نوع التوصيل</option></select>

  <label>البلدية</label>
  <input id="commune">

  <div class="summary">
    <div><span>سعر التوصيل</span><span><span id="delivery">0</span> دج</span></div>
    <div class="total"><span>الإجمالي</span><span id="total"></span> دج</div>
  </div>

  <button id="orderBtn">تأكيد الطلب</button>
</div>

<div class="modal" id="confirmModal">
  <div class="modal-box">
    <div id="orderSummary"></div>
    <div class="modal-actions">
      <button class="confirm" id="confirmSend">إرسال واتساب</button>
      <button class="cancel" onclick="confirmModal.style.display='none'">إلغاء</button>
    </div>
  </div>
</div>

<script>
var productSelect=document.getElementById("productSelect"),
productImg=document.getElementById("productImg"),
productPrice=document.getElementById("productPrice"),
qty=document.getElementById("qty"),
wilaya=document.getElementById("wilaya"),
deliveryType=document.getElementById("deliveryType"),
deliverySpan=document.getElementById("delivery"),
totalSpan=document.getElementById("total"),
nameInput=document.getElementById("name"),
phoneInput=document.getElementById("phone"),
commune=document.getElementById("commune");

var delivery=0,extra=0,currentProduct=PRODUCTS[0];

// تعبئة قائمة المنتجات
PRODUCTS.forEach((p,i)=>{
  let o=document.createElement("option");
  o.value=i; o.textContent=p.name;
  productSelect.appendChild(o);
});

// تعبئة قائمة الولايات
for(let w in SETTINGS.deliveryPrices){
  let o=document.createElement("option");
  o.value=w; o.textContent=w;
  wilaya.appendChild(o);
}

// تعبئة قائمة نوع التوصيل
for(let t in SETTINGS.deliveryTypes){
  let o=document.createElement("option");
  o.value=t; o.textContent=t;
  deliveryType.appendChild(o);
}

// تحديث المنتج
function updateProduct(){
  currentProduct=PRODUCTS[productSelect.value];
  productImg.src=currentProduct.image;
  productPrice.innerText=currentProduct.price+" دج";
  updateTotal();
}
productSelect.onchange=updateProduct;
updateProduct();

// تحديث الإجمالي
function updateTotal(){
  totalSpan.innerText=(currentProduct.price*qty.value)+delivery+extra;
}
plus.onclick=()=>{qty.value++;updateTotal();}
minus.onclick=()=>{if(qty.value>1){qty.value--;updateTotal();}}

// حساب التوصيل
wilaya.onchange=function(){
  delivery=SETTINGS.deliveryPrices[this.value]||0;
  deliverySpan.innerText=delivery+extra;
  updateTotal();
};
deliveryType.onchange=function(){
  extra=SETTINGS.deliveryTypes[this.value]||0;
  deliverySpan.innerText=delivery+extra;
  updateTotal();
};

// زر تأكيد الطلب
orderBtn.onclick=function(){
  if(!nameInput.value||!phoneInput.value||!wilaya.value||!commune.value||!deliveryType.value){
    alert("عمر جميع الحقول"); return;
  }
  orderSummary.innerHTML=`
  <p>المنتج: ${currentProduct.name}</p>
  <p>الكمية: ${qty.value}</p>
  <p>الولاية: ${wilaya.value}</p>
  <p>البلدية: ${commune.value}</p>
  <p><b>الإجمالي: ${totalSpan.innerText} دج</b></p>`;
  confirmModal.style.display="flex";
};

// إرسال الطلب عبر واتساب
confirmSend.onclick=function(){
  var msg=`طلب جديد:
الاسم: ${nameInput.value}
الهاتف: ${phoneInput.value}
المنتج: ${currentProduct.name}
الكمية: ${qty.value}
الولاية: ${wilaya.value}
البلدية: ${commune.value}
الإجمالي: ${totalSpan.innerText} دج`;
  window.open("https://wa.me/"+SETTINGS.whatsapp+"?text="+encodeURIComponent(msg));
};
</script>

</body>
</html>
