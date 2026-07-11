<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Siparişli Mobil Menü & Akıllı Asistan</title>
    <style>
        :root {
            --bg-dark: #121212;
            --surface-dark: #1e1e1e;
            --surface-active-green: #1b4d22;
            --primary-accent: #e67e22;
            --success-color: #2ecc71;
            --text-main: #ffffff;
            --text-muted: #aaaaaa;
            --bot-bg: #2c3e50;
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            margin: 0; padding: 0 0 90px 0; background-color: var(--bg-dark); color: var(--text-main);
        }

        header {
            background: linear-gradient(rgba(0,0,0,0.7), rgba(18,18,18,1)), url('https://images.unsplash.com/photo-1555396273-367ea4eb4db5?q=80&w=500') center/cover;
            padding: 3rem 1.5rem 2rem 1.5rem; text-align: center; border-bottom: 1px solid #2c2c2c;
        }
        header h1 { margin: 0; font-size: 2.2rem; font-weight: 800; }
        header p { margin: 0.5rem 0 0 0; color: var(--primary-accent); font-size: 0.95rem; }

        .container { padding: 1rem; }
        .category-section { scroll-margin-top: 20px; margin-bottom: 2.5rem; }
        .category-title { font-size: 1.4rem; font-weight: 700; padding-left: 0.5rem; border-left: 4px solid var(--primary-accent); margin-bottom: 1rem; }

        /* Yemek Kartı */
        .menu-item {
            display: flex; justify-content: space-between; align-items: center;
            background-color: var(--surface-dark); padding: 1rem; margin-bottom: 0.8rem;
            border-radius: 12px; border: 1px solid #2a2a2a;
            transition: all 0.3s ease;
        }
        
        .menu-item.active-in-cart {
            background-color: var(--surface-active-green);
            border-color: var(--success-color);
        }

        .item-details { flex: 1; padding-right: 1rem; }
        .item-name { font-size: 1.1rem; font-weight: 600; margin: 0 0 0.3rem 0; }
        .item-desc { font-size: 0.85rem; color: var(--text-muted); margin: 0; }
        
        .item-right { display: flex; flex-direction: column; align-items: flex-end; gap: 0.6rem; }
        .item-price { font-size: 1.1rem; font-weight: 700; color: var(--primary-accent); }
        .item-unit { font-size: 0.8rem; color: var(--text-muted); font-weight: normal; }
        
        .quantity-control {
            display: flex; align-items: center; background: rgba(0,0,0,0.3); 
            border-radius: 20px; padding: 2px; border: 1px solid #444;
        }
        .qty-btn {
            background: var(--primary-accent); color: white; border: none;
            width: 28px; height: 28px; border-radius: 50%; font-size: 1.1rem;
            font-weight: bold; cursor: pointer; display: flex; justify-content: center; align-items: center;
        }
        .qty-btn.minus { background: #555; }
        .qty-btn:active { opacity: 0.7; }
        .qty-text { padding: 0 10px; font-weight: 600; font-size: 0.95rem; min-width: 25px; text-align: center; }

        /* Yüzen Butonlar */
        .floating-cart-btn {
            position: fixed; bottom: 155px; right: 20px; background-color: var(--success-color);
            color: white; border: none; width: 60px; height: 60px; border-radius: 50%;
            font-size: 1.5rem; display: flex; justify-content: center; align-items: center;
            box-shadow: 0 4px 10px rgba(0,0,0,0.3); z-index: 999; cursor: pointer;
        }
        .cart-count {
            position: absolute; top: 0; right: 0; background-color: red;
            color: white; font-size: 0.75rem; width: 20px; height: 20px;
            border-radius: 50%; display: flex; justify-content: center; align-items: center;
        }

        .floating-bot-btn {
            position: fixed; bottom: 85px; right: 20px; background-color: #3498db;
            color: white; border: none; width: 60px; height: 60px; border-radius: 50%;
            font-size: 1.5rem; display: flex; justify-content: center; align-items: center;
            box-shadow: 0 4px 10px rgba(0,0,0,0.3); z-index: 999; cursor: pointer;
        }

        /* Sepet Sayfası (Modal) */
        .cart-modal {
            position: fixed; top: 0; left: 0; right: 0; bottom: 0;
            background-color: var(--bg-dark); z-index: 1000; display: none;
            padding: 2rem 1.5rem; overflow-y: auto;
        }
        .close-cart { font-size: 2rem; color: var(--text-muted); cursor: pointer; float: right; }
        .cart-title { font-size: 1.8rem; margin-bottom: 1.5rem; }
        
        .form-group { margin-bottom: 1.2rem; }
        .form-group label { display: block; margin-bottom: 0.4rem; color: var(--text-muted); font-size: 0.9rem; }
        .form-group input, .form-group textarea, .form-group select {
            width: 100%; padding: 0.8rem; background-color: var(--surface-dark);
            border: 1px solid #333; color: white; border-radius: 8px; font-size: 1rem;
        }
        .submit-btn {
            background-color: var(--success-color); color: white; border: none; width: 100%;
            padding: 1rem; border-radius: 8px; font-size: 1.1rem; font-weight: 700; cursor: pointer; margin-top: 1rem;
        }
        .cart-items-list { margin-bottom: 1.5rem; background: var(--surface-dark); padding: 1rem; border-radius: 8px; }
        .cart-summary { font-size: 1.2rem; font-weight: 700; text-align: right; margin-bottom: 1.5rem; color: var(--primary-accent); }

        .iban-info-box {
            background-color: #2c3e50; border: 1px solid #34495e; padding: 1rem;
            border-radius: 8px; margin-bottom: 1.2rem; display: none;
        }
        .iban-info-box p { margin: 0.3rem 0; font-size: 0.9rem; color: #ecf0f1; }
        .iban-number { font-family: monospace; font-size: 1.05rem; color: #f1c40f; font-weight: bold; letter-spacing: 0.5px; }

        /* CHATBOT ARALIK PANELİ */
        .bot-modal {
            position: fixed; bottom: 85px; right: 20px; width: 340px; height: 450px;
            background-color: var(--surface-dark); border: 2px solid #333; border-radius: 16px;
            z-index: 1001; display: none; flex-direction: column; overflow: hidden;
            box-shadow: 0 8px 25px rgba(0,0,0,0.5);
        }
        .bot-header { background-color: #3498db; padding: 0.8rem 1rem; display: flex; justify-content: space-between; align-items: center; }
        .bot-header h3 { margin: 0; font-size: 1rem; }
        .bot-close { cursor: pointer; font-size: 1.2rem; font-weight: bold; }
        .bot-messages { flex: 1; padding: 1rem; overflow-y: auto; display: flex; flex-direction: column; gap: 0.8rem; }
        .msg { max-width: 80%; padding: 0.6rem 0.8rem; border-radius: 12px; font-size: 0.9rem; line-height: 1.4; white-space: pre-line; }
        .msg.bot { background-color: var(--bot-bg); align-self: flex-start; border-bottom-left-radius: 2px; }
        .msg.user { background-color: var(--primary-accent); align-self: flex-end; border-bottom-right-radius: 2px; }
        
        /* HAZIR SEÇENEK BUTONLARI */
        .bot-options-area {
            display: flex; flex-direction: column; gap: 0.5rem; padding: 0.8rem;
            background: #1a1a1a; border-top: 1px solid #333;
        }
        .bot-opt-btn {
            background-color: #34495e; color: white; border: 1px solid #4f5d75;
            padding: 0.7rem; border-radius: 8px; font-size: 0.85rem; font-weight: 600;
            cursor: pointer; text-align: left; transition: background 0.2s;
        }
        .bot-opt-btn:hover { background-color: #2c3e50; }
        .bot-opt-btn:active { background-color: var(--primary-accent); }
        
        /* Mobil Kaydırılabilir Alt Navigasyon */
        .bottom-nav {
            position: fixed; bottom: 0; left: 0; right: 0; height: 65px;
            background-color: #1a1a1a; border-top: 1px solid #2c2c2c;
            display: flex; justify-content: flex-start; align-items: center; z-index: 999;
            overflow-x: auto; white-space: nowrap; padding: 0 1rem; gap: 1rem;
        }
        .bottom-nav::-webkit-scrollbar { display: none; }
        .nav-link { color: var(--text-muted); text-decoration: none; font-size: 0.85rem; font-weight: 600; background: #2a2a2a; padding: 0.4rem 0.8rem; border-radius: 20px; }
    </style>
</head>
<body>

    <header>
        <h1>Ev Lezzetleri Mutfak</h1>
        <p>Özel Sipariş & Dijital Menü</p>
    </header>

    <!-- Sipariş Sepet Butonu -->
    <button class="floating-cart-btn" onclick="toggleCart(true)">
        🛒 <span class="cart-count" id="cart-count">0</span>
    </button>

    <!-- Yapay Zeka Bot Butonu -->
    <button class="floating-bot-btn" onclick="toggleBot(true)">
        🤖
    </button>

    <div class="container">
        
        <!-- KATEGORİ: ANA YEMEKLER -->
        <div class="category-section" id="anayemekler">
            <div class="category-title">Ana Yemekler</div>
            
            <div class="menu-item" id="item-manti">
                <div class="item-details">
                    <h3 class="item-name">Ev Yapımı Mantı</h3>
                    <p class="item-desc">Özel sosu, yoğurdu ve kızgın tereyağı ile enfes sunum.</p>
                </div>
                <div class="item-right">
                    <span class="item-price">500 TL <span class="item-unit">/ Kg</span></span>
                    <div class="quantity-control">
                        <button class="qty-btn minus" onclick="changeQuantity('item-manti', 'Ev Yapımı Mantı', 500, -1, 'Kg')">-</button>
                        <span class="qty-text" id="qty-item-manti">0</span>
                        <button class="qty-btn" onclick="changeQuantity('item-manti', 'Ev Yapımı Mantı', 500, 1, 'Kg')">+</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- KATEGORİ: MEZE & APERATİFLER -->
        <div class="category-section" id="meze-aperatif">
            <div class="category-title">Meze & Aperatifler</div>
            
            <div class="menu-item" id="item-sarma">
                <div class="item-details">
                    <h3 class="item-name">Yaprak Sarma</h3>
                    <p class="item-desc">İncecik sarılmış, zeytinyağlı ve limon dilimli ev sarması.</p>
                </div>
                <div class="item-right">
                    <span class="item-price">500 TL <span class="item-unit">/ Kg</span></span>
                    <div class="quantity-control">
                        <button class="qty-btn minus" onclick="changeQuantity('item-sarma', 'Yaprak Sarma', 500, -1, 'Kg')">-</button>
                        <span class="qty-text" id="qty-item-sarma">0</span>
                        <button class="qty-btn" onclick="changeQuantity('item-sarma', 'Yaprak Sarma', 500, 1, 'Kg')">+</button>
                    </div>
                </div>
            </div>

            <div class="menu-item" id="item-iclikofte">
                <div class="item-details">
                    <h3 class="item-name">İçli Köfte</h3>
                    <p class="item-desc">Bol malzemeli iç harcı ve tam kıvamında dış kabuğuyla sıcak servis.</p>
                </div>
                <div class="item-right">
                    <span class="item-price">500 TL <span class="item-unit">/ Adet</span></span>
                    <div class="quantity-control">
                        <button class="qty-btn minus" onclick="changeQuantity('item-iclikofte', 'İçli Köfte', 500, -1, 'Adet')">-</button>
                        <span class="qty-text" id="qty-item-iclikofte">0</span>
                        <button class="qty-btn" onclick="changeQuantity('item-iclikofte', 'İçli Köfte', 500, 1, 'Adet')">+</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- KATEGORİ: BÖREKLER -->
        <div class="category-section" id="borekler">
            <div class="category-title">Katmer Börekler</div>
            
            <div class="menu-item" id="item-borek-patates">
                <div class="item-details">
                    <h3 class="item-name">Patatesli Katmer Börek</h3>
                    <p class="item-desc">Çıtır çıtır kat kat açılmış, özel baharatlı patates harçlı.</p>
                </div>
                <div class="item-right">
                    <span class="item-price">500 TL <span class="item-unit">/ Kg</span></span>
                    <div class="quantity-control">
                        <button class="qty-btn minus" onclick="changeQuantity('item-borek-patates', 'Patatesli Katmer Börek', 500, -1, 'Kg')">-</button>
                        <span class="qty-text" id="qty-item-borek-patates">0</span>
                        <button class="qty-btn" onclick="changeQuantity('item-borek-patates', 'Patatesli Katmer Börek', 500, 1, 'Kg')">+</button>
                    </div>
                </div>
            </div>

            <div class="menu-item" id="item-borek-peynir">
                <div class="item-details">
                    <h3 class="item-name">Peynirli Katmer Börek</h3>
                    <p class="item-desc">Özel peynir ve maydanoz karışımıyla harmanlanmış katmer lezzeti.</p>
                </div>
                <div class="item-right">
                    <span class="item-price">500 TL <span class="item-unit">/ Kg</span></span>
                    <div class="quantity-control">
                        <button class="qty-btn minus" onclick="changeQuantity('item-borek-peynir', 'Peynirli Katmer Börek', 500, -1, 'Kg')">-</button>
                        <span class="qty-text" id="qty-item-borek-peynir">0</span>
                        <button class="qty-btn" onclick="changeQuantity('item-borek-peynir', 'Peynirli Katmer Börek', 500, 1, 'Kg')">+</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- KATEGORİ: TATLILAR -->
        <div class="category-section" id="tatlilar">
            <div class="category-title">Tatlı & Kurabiyeler</div>
            
            <div class="menu-item" id="item-kurabiye">
                <div class="item-details">
                    <h3 class="item-name">Elmalı Kurabiye</h3>
                    <p class="item-desc">Ağızda dağılan kıyır kıyır hamuru, tarçınlı ve elmalı nefis iç harcıyla.</p>
                </div>
                <div class="item-right">
                    <span class="item-price">500 TL <span class="item-unit">/ Kg</span></span>
                    <div class="quantity-control">
                        <button class="qty-btn minus" onclick="changeQuantity('item-kurabiye', 'Elmalı Kurabiye', 500, -1, 'Kg')">-</button>
                        <span class="qty-text" id="qty-item-kurabiye">0</span>
                        <button class="qty-btn" onclick="changeQuantity('item-kurabiye', 'Elmalı Kurabiye', 500, 1, 'Kg')">+</button>
                    </div>
                </div>
            </div>

            <div class="menu-item" id="item-cheesecake">
                <div class="item-details">
                    <h3 class="item-name">Cheesecake</h3>
                    <p class="item-desc">Yoğun kremalı, tabanı çıtır bisküvili, nefis sos seçeneğiyle ev yapımı cheesecake.</p>
                </div>
                <div class="item-right">
                    <span class="item-price">500 TL <span class="item-unit">/ Kg</span></span>
                    <div class="quantity-control">
                        <button class="qty-btn minus" onclick="changeQuantity('item-cheesecake', 'Cheesecake', 500, -1, 'Kg')">-</button>
                        <span class="qty-text" id="qty-item-cheesecake">0</span>
                        <button class="qty-btn" onclick="changeQuantity('item-cheesecake', 'Cheesecake', 500, 1, 'Kg')">+</button>
                    </div>
                </div>
            </div>
        </div>

    </div>

    <!-- SEPET MODAL EKRANI -->
    <div class="cart-modal" id="cart-modal">
        <span class="close-cart" onclick="toggleCart(false)">&times;</span>
        <h2 class="cart-title">Siparişiniz</h2>
        
        <div class="cart-items-list" id="cart-items-list">
            Sepetiniz boş.
        </div>
        <div class="cart-summary" id="cart-summary">Toplam: 0 TL</div>

        <!-- FORMSPREE DIRECT EMAIL ALTYAPISI -->
        <form action="https://formspree.io/f/xoqgkyrj" method="POST" id="order-form">
            <!-- Gerekli Gizli Post Verileri (Gmail'e Gitmesi İçin) -->
            <input type="hidden" name="Sipariş Detayları" id="hidden-cart-data">
            <input type="hidden" name="Toplam Tutar" id="hidden-total-data">
            <input type="hidden" name="Bu Cihazdan Toplam Sipariş Sayısı" id="hidden-order-count-data">
            
            <!-- Formspree Hedef Gmail Yönlendirmesi -->
            <input type="hidden" name="_to" value="mcakiroglu006@gmail.com">

            <div class="form-group">
                <label>Adınız Soyadınız</label>
                <input type="text" name="Müşteri Adı" required placeholder="Ahmet Yılmaz">
            </div>
            <div class="form-group">
                <label>Telefon Numaranız</label>
                <input type="tel" name="Telefon" required placeholder="05xx xxx xx xx">
            </div>
            <div class="form-group">
                <label>Teslimat Adresi / Masa No</label>
                <textarea name="Adres veya Masa" rows="3" required placeholder="Masa No veya Ev Adresi girin"></textarea>
            </div>
            
            <div class="form-group">
                <label>Ödeme Yöntemi</label>
                <select name="Ödeme Tipi" id="payment-method" onchange="checkPaymentMethod()">
                    <option value="Kapıda Nakit">Kapıda Nakit</option>
                    <option value="Kapıda Kredi Kartı">Kapıda Kredi Kartı</option>
                    <option value="IBAN'a Gönder (Havale/EFT)">IBAN'a Gönder (Havale/EFT)</option>
                </select>
            </div>

            <div class="iban-info-box" id="iban-info-box">
                <p><strong>Banka:</strong> Akbank</p>
                <p><strong>Alıcı:</strong> İşletme Adı</p>
                <p><strong>IBAN:</strong> <span class="iban-number">TR99 0004 6000 1234 5678 9012</span></p>
            </div>

            <button type="submit" class="submit-btn">Siparişi Tamamla & Gönder</button>
        </form>
    </div>

    <!-- YAPAY ZEKA SOHBET MODAL PANELİ -->
    <div class="bot-modal" id="bot-modal">
        <div class="bot-header">
            <h3>Mutfak Asistanı 🤖</h3>
            <span class="bot-close" onclick="toggleBot(false)">&times;</span>
        </div>
        <div class="bot-messages" id="bot-messages">
            <div class="msg bot">Merhaba! Ben dijital mutfak asistanınız. Bilgi almak istediğiniz konuyu aşağıdaki seçeneklerden seçebilirsiniz:</div>
        </div>
        
        <!-- MÜŞTERİNİN SEÇECEĞİ HAZIR İKİ SORU BUTONU -->
        <div class="bot-options-area">
            <button class="bot-opt-btn" onclick="selectBotOption('delivery')">🕒 Siparişim ne zaman teslim edilir?</button>
            <button class="bot-opt-btn" onclick="selectBotOption('history')">📦 Geçmiş siparişlerim neler?</button>
        </div>
    </div>

    <!-- ALT NAVİGASYON -->
    <div class="bottom-nav">
        <a href="#anayemekler" class="nav-link">🍖 Ana Yemekler</a>
        <a href="#meze-aperatif" class="nav-link">🥗 Meze & Aperatif</a>
        <a href="#borekler" class="nav-link">🥮 Katmer Börekler</a>
        <a href="#tatlilar" class="nav-link">🍰 Tatlı & Kurabiye</a>
    </div>

    <script>
        let cart = {};

        // --- CHATBOT SEÇENEK FONKSİYONLARI ---
        function toggleBot(show) {
            document.getElementById('bot-modal').style.display = show ? 'flex' : 'none';
        }

        function selectBotOption(optionType) {
            if (optionType === 'delivery') {
                appendMessage("Siparişim ne zaman teslim edilir?", 'user');
                setTimeout(() => {
                    appendMessage("Sipariş teslimatı, sipariş verildikten sonraki 1 ila 5 gün arası değişmektedir. 🕒", 'bot');
                }, 400);
            } 
            else if (optionType === 'history') {
                appendMessage("Geçmiş siparişlerim neler?", 'user');
                setTimeout(() => {
                    let savedOrders = localStorage.getItem('order_history');
                    if (savedOrders) {
                        let ordersArray = JSON.parse(savedOrders);
                        appendMessage("O zamana kadar verdiğiniz tüm siparişler:\n\n" + ordersArray.join("\n\n"), 'bot');
                    } else {
                        appendMessage("Henüz bu cihazdan kayıtlı geçmiş bir siparişiniz bulunmamaktadır. 📦", 'bot');
                    }
                }, 400);
            }
        }

        function appendMessage(text, sender) {
            const msgDiv = document.createElement('div');
            msgDiv.classList.add('msg', sender);
            msgDiv.innerText = text;
            const container = document.getElementById('bot-messages');
            container.appendChild(msgDiv);
            container.scrollTop = container.scrollHeight;
        }

        // --- SEPET VE SİPARİŞ FONKSİYONLARI ---
        function checkPaymentMethod() {
            const method = document.getElementById('payment-method').value;
            const ibanBox = document.getElementById('iban-info-box');
            if (method.includes("IBAN")) {
                ibanBox.style.display = 'block';
            } else {
                ibanBox.style.display = 'none';
            }
        }

        function changeQuantity(id, name, price, amount, unit) {
            if (!cart[id]) {
                cart[id] = { name: name, price: price, quantity: 0, unit: unit };
            }

            cart[id].quantity += amount;

            if (cart[id].quantity <= 0) {
                delete cart[id];
                document.getElementById('qty-' + id).innerText = 0;
                document.getElementById(id).classList.remove('active-in-cart');
            } else {
                document.getElementById('qty-' + id).innerText = Math.max(0, cart[id].quantity);
                document.getElementById(id).classList.add('active-in-cart');
            }

            updateCartUI();
        }

        function toggleCart(show) {
            document.getElementById('cart-modal').style.display = show ? 'block' : 'none';
            if (show) checkPaymentMethod();
        }

        function updateCartUI() {
            const list = document.getElementById('cart-items-list');
            const summary = document.getElementById('cart-summary');
            
            let totalItems = 0;
            let totalPrice = 0;
            let html = "";
            let emailText = "";

            for (let id in cart) {
                let item = cart[id];
                let itemTotal = item.price * item.quantity;
                totalPrice += itemTotal;
                totalItems += item.quantity;

                html += `<div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:0.8rem; border-bottom:1px dashed #444; padding-bottom:0.5rem;">
                            <div>
                                <strong>${item.name}</strong> <br>
                                <span style="color:var(--primary-accent)">${item.price} TL x ${item.quantity} ${item.unit}</span>
                            </div>
                            <div class="quantity-control">
                                <button class="qty-btn minus" onclick="changeQuantity('${id}', '${item.name}', ${item.price}, -1, '${item.unit}')">-</button>
                                <span class="qty-text">${item.quantity}</span>
                                <button class="qty-btn" onclick="changeQuantity('${id}', '${item.name}', ${item.price}, 1, '${item.unit}')">+</button>
                            </div>
                         </div>`;

                emailText += `${item.name} -> ${item.quantity} ${item.unit} (${itemTotal} TL), `;
            }

            document.getElementById('cart-count').innerText = totalItems;

            if (totalItems === 0) {
                list.innerHTML = "Sepetiniz boş.";
                summary.innerText = "Toplam: 0 TL";
                return;
            }

            list.innerHTML = html;
            summary.innerText = `Toplam: ${totalPrice} TL`;
            
            document.getElementById('hidden-cart-data').value = emailText;
            document.getElementById('hidden-total-data').value = totalPrice + " TL";
        }

        // SİPARİŞ KAYIT VE GMAIL GÖNDERİM MEKANİZMASI
        document.getElementById('order-form').onsubmit = function(e) {
            let existingOrders = localStorage.getItem('order_history');
            let ordersArray = existingOrders ? JSON.parse(existingOrders) : [];
            
            let currentOrderNumber = ordersArray.length + 1;
            document.getElementById('hidden-order-count-data').value = currentOrderNumber + ". Siparişi";

            let currentOrderDetails = document.getElementById('hidden-cart-data').value;
            let currentTotal = document.getElementById('hidden-total-data').value;
            let currentDate = new Date().toLocaleString('tr-TR');

            let newOrderLog = `[Sipariş No: #${currentOrderNumber} - Tarih: ${currentDate}]\nÜrünler: ${currentOrderDetails}\nTutar: ${currentTotal}`;
            ordersArray.push(newOrderLog);
            localStorage.setItem('order_history', JSON.stringify(ordersArray));
        };
    </script>
</body>
</html>
