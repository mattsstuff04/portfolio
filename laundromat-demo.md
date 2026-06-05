---
layout: default
title: Jb's Laundry Demo
---

<div class="laundromat-demo">
    <!-- laundromat HTML code from the previous answer here -->
    <nav class="navbar">
    <div class="logo">
        <h2><i class="fas fa-tshirt"></i> Jb's Laundry</h2>
        <p>Fresh & Clean • Chicago's Best</p>
    </div>
    <div class="nav-links" id="navLinks">
    </div>
</nav>

<div class="container" id="mainContent">
    <!-- Homepage / Dashboard -->
</div>

<footer>
    <p>📍 1500 W Howard St, Chicago, IL 60626 | ☎️ (773) 555-0123 | ✉️ hello@jblaundry.com</p>
    <p>&copy; 2025 Jb's Laundry - Monthly Wash Plans</p>
</footer>

<script>
    // ---------- STORAGE KEYS ----------
    const STORAGE_USERS = "jbLaundry_users";
    const STORAGE_CURRENT_USER = "jbLaundry_currentUser";
    const STORAGE_SUBSCRIPTIONS = "jbLaundry_subs";

    // Pricing plans
    const PLANS = {
        basic: { name: "Basic Wash", price: 29.99, features: ["10 wash loads/month", "Basic detergent", "Dryer credit $5"] },
        standard: { name: "Standard Wash", price: 49.99, features: ["25 wash loads/month", "Premium detergent", "Dryer credit $15", "Fold service (2x/mo)"] },
        premium: { name: "Premium Wash", price: 89.99, features: ["Unlimited loads", "Eco detergent", "Dryer unlimited", "Free pick-up & drop-off", "Priority service"] }
    };

    // Helper: load users
    function getUsers() {
        let users = localStorage.getItem(STORAGE_USERS);
        if (!users) {
            const defaultUsers = [
                { email: "demo@jb.com", password: "demo123", name: "Demo User" }
            ];
            localStorage.setItem(STORAGE_USERS, JSON.stringify(defaultUsers));
            return defaultUsers;
        }
        return JSON.parse(users);
    }

    function saveUsers(users) {
        localStorage.setItem(STORAGE_USERS, JSON.stringify(users));
    }

    function getCurrentUser() {
        return localStorage.getItem(STORAGE_CURRENT_USER);
    }

    function setCurrentUser(email) {
        if (email) localStorage.setItem(STORAGE_CURRENT_USER, email);
        else localStorage.removeItem(STORAGE_CURRENT_USER);
    }

    function getSubscriptions() {
        let subs = localStorage.getItem(STORAGE_SUBSCRIPTIONS);
        if (!subs) return {};
        return JSON.parse(subs);
    }

    function saveSubscriptions(subs) {
        localStorage.setItem(STORAGE_SUBSCRIPTIONS, JSON.stringify(subs));
    }

    function getUserSubscription(email) {
        const subs = getSubscriptions();
        return subs[email] || null;
    }

    function setUserSubscription(email, planId, status = "active", startDate = new Date().toISOString()) {
        const subs = getSubscriptions();
        subs[email] = { planId, status, startDate, nextBilling: new Date(Date.now() + 30*86400000).toISOString() };
        saveSubscriptions(subs);
    }

    function cancelSubscription(email) {
        const subs = getSubscriptions();
        if (subs[email]) {
            subs[email].status = "cancelled";
            saveSubscriptions(subs);
        }
    }

    // UI Rendering
    function renderNav() {
        const navDiv = document.getElementById("navLinks");
        const currentEmail = getCurrentUser();
        if (currentEmail) {
            const users = getUsers();
            const user = users.find(u => u.email === currentEmail);
            const displayName = user ? user.name : currentEmail.split('@')[0];
            navDiv.innerHTML = `
                <span style="margin-right: 10px;">👋 Hello, ${displayName}</span>
                <button id="dashboardBtn" class="btn-outline">Dashboard</button>
                <button id="logoutBtn" class="btn-outline">Logout</button>
            `;
            document.getElementById("dashboardBtn")?.addEventListener("click", () => showDashboard());
            document.getElementById("logoutBtn")?.addEventListener("click", logout);
        } else {
            navDiv.innerHTML = `
                <button id="loginNavBtn" class="btn-outline">Login</button>
                <button id="registerNavBtn" class="btn-primary">Sign Up</button>
            `;
            document.getElementById("loginNavBtn")?.addEventListener("click", () => showLoginForm());
            document.getElementById("registerNavBtn")?.addEventListener("click", () => showRegisterForm());
        }
    }

    function showHomepage() {
        const container = document.getElementById("mainContent");
        container.innerHTML = `
            <div class="hero">
                <h1><i class="fas fa-soap"></i> Jb's Laundry</h1>
                <div class="hero-address"><i class="fas fa-map-marker-alt"></i> 1500 W Howard St, Chicago, IL 60626</div>
                <p>Clean clothes, happy life. Monthly unlimited wash plans starting at $29.99. Sign up today and never worry about laundry again!</p>
                <p><i class="fas fa-clock"></i> Open daily: 7am - 10pm | <i class="fas fa-wifi"></i> Free WiFi</p>
            </div>
            <h2 style="text-align: center; margin: 1rem 0;">💧 Choose Your Monthly Wash Plan</h2>
            <div class="plans" id="plansContainer"></div>
        `;
        const plansDiv = document.getElementById("plansContainer");
        plansDiv.innerHTML = Object.keys(PLANS).map(planId => {
            const p = PLANS[planId];
            return `
                <div class="card">
                    <h3>${p.name}</h3>
                    <div class="price">$${p.price}<span>/month</span></div>
                    <ul>
                        ${p.features.map(f => `<li><i class="fas fa-check-circle"></i> ${f}</li>`).join('')}
                    </ul>
                    <button class="subscribe-btn" data-plan="${planId}">Subscribe Now</button>
                </div>
            `;
        }).join('');
        // attach subscribe events
        document.querySelectorAll('.subscribe-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                const planId = btn.getAttribute('data-plan');
                handleSubscribe(planId);
            });
        });
    }

    function handleSubscribe(planId) {
        const currentEmail = getCurrentUser();
        if (!currentEmail) {
            alert("Please login or create an account first.");
            showLoginForm();
            return;
        }
        const plan = PLANS[planId];
        if (!plan) return;
        // Simulate payment flow (stripe demo alert)
        if (confirm(`You are subscribing to ${plan.name} for $${plan.price}/month.\n\nClick OK to proceed to payment (demo mode).\nIn production, Stripe would process the card.`)) {
            // mock payment success
            setUserSubscription(currentEmail, planId, "active");
            alert(`✅ Success! You are now subscribed to ${plan.name}. Welcome to Jb's Laundry!`);
            showDashboard();
        }
    }

    function showDashboard() {
        const currentEmail = getCurrentUser();
        if (!currentEmail) {
            showLoginForm();
            return;
        }
        const subscription = getUserSubscription(currentEmail);
        const container = document.getElementById("mainContent");
        let subHtml = "";
        if (subscription && subscription.status === "active") {
            const plan = PLANS[subscription.planId];
            subHtml = `
                <div class="subscription-card">
                    <div><strong>${plan.name}</strong> - $${plan.price}/month</div>
                    <div><span class="status-active">ACTIVE</span></div>
                    <div>Next billing: ${new Date(subscription.nextBilling).toLocaleDateString()}</div>
                    <button id="cancelSubBtn" class="small-btn">Cancel Plan</button>
                </div>
            `;
        } else if (subscription && subscription.status === "cancelled") {
            subHtml = `
                <div class="subscription-card">
                    <div><strong>Previous plan: ${PLANS[subscription.planId]?.name}</strong></div>
                    <div><span class="status-cancelled">CANCELLED</span></div>
                    <div>Your membership ended. <button id="resubBtn" class="small-btn">Re-subscribe</button></div>
                </div>
            `;
        } else {
            subHtml = `<div style="background:#ffe8e0; padding:1rem; border-radius: 20px;">❌ No active subscription. <a href="#" id="browsePlansLink">Browse our monthly plans</a></div>`;
        }

        container.innerHTML = `
            <div class="dashboard">
                <h2><i class="fas fa-user-circle"></i> My Dashboard</h2>
                <p>Welcome back, ${currentEmail}!</p>
                <h3>📋 Current Subscription</h3>
                ${subHtml}
                <hr style="margin: 1.5rem 0;">
                <h3>🔄 Payment History (demo)</h3>
                <p>Your last payment was successfully processed via Stripe test mode.</p>
                <button id="homeFromDashBtn" class="small-btn" style="background:#2a9d8f;">🏠 Back to Homepage</button>
            </div>
        `;
        if (document.getElementById("cancelSubBtn")) {
            document.getElementById("cancelSubBtn").addEventListener("click", () => {
                if (confirm("Cancel your monthly washing plan?")) {
                    cancelSubscription(currentEmail);
                    alert("Subscription cancelled. You can resubscribe anytime.");
                    showDashboard();
                }
            });
        }
        if (document.getElementById("resubBtn")) {
            document.getElementById("resubBtn").addEventListener("click", () => showHomepage());
        }
        if (document.getElementById("browsePlansLink")) {
            document.getElementById("browsePlansLink").addEventListener("click", (e) => {
                e.preventDefault();
                showHomepage();
            });
        }
        if (document.getElementById("homeFromDashBtn")) {
            document.getElementById("homeFromDashBtn").addEventListener("click", () => showHomepage());
        }
    }

    function showLoginForm() {
        const container = document.getElementById("mainContent");
        container.innerHTML = `
            <div class="form-box">
                <h2>Login to Jb's Laundry</h2>
                <input type="email" id="loginEmail" placeholder="Email address" autocomplete="email">
                <input type="password" id="loginPassword" placeholder="Password">
                <button id="doLoginBtn">Login</button>
                <p style="margin-top:1rem;">Don't have an account? <a href="#" id="gotoRegister">Sign up</a></p>
                <p style="font-size:0.8rem; margin-top:1rem;">Demo: demo@jb.com / demo123</p>
            </div>
        `;
        document.getElementById("doLoginBtn").addEventListener("click", () => {
            const email = document.getElementById("loginEmail").value.trim();
            const pass = document.getElementById("loginPassword").value;
            const users = getUsers();
            const user = users.find(u => u.email === email && u.password === pass);
            if (user) {
                setCurrentUser(email);
                renderNav();
                showDashboard();
            } else {
                alert("Invalid credentials. Try demo@jb.com / demo123");
            }
        });
        document.getElementById("gotoRegister")?.addEventListener("click", (e) => {
            e.preventDefault();
            showRegisterForm();
        });
    }

    function showRegisterForm() {
        const container = document.getElementById("mainContent");
        container.innerHTML = `
            <div class="form-box">
                <h2>Create Account</h2>
                <input type="text" id="regName" placeholder="Full Name">
                <input type="email" id="regEmail" placeholder="Email address">
                <input type="password" id="regPassword" placeholder="Password">
                <button id="doRegisterBtn">Sign Up</button>
                <p>Already have an account? <a href="#" id="gotoLogin">Login</a></p>
            </div>
        `;
        document.getElementById("doRegisterBtn").addEventListener("click", () => {
            const name = document.getElementById("regName").value.trim();
            const email = document.getElementById("regEmail").value.trim();
            const password = document.getElementById("regPassword").value;
            if (!name || !email || !password) {
                alert("Please fill all fields");
                return;
            }
            const users = getUsers();
            if (users.find(u => u.email === email)) {
                alert("User already exists. Please login.");
                showLoginForm();
                return;
            }
            users.push({ email, password, name });
            saveUsers(users);
            setCurrentUser(email);
            renderNav();
            alert("Account created! You can now subscribe to a plan.");
            showHomepage();
        });
        document.getElementById("gotoLogin")?.addEventListener("click", (e) => {
            e.preventDefault();
            showLoginForm();
        });
    }

    function logout() {
        setCurrentUser(null);
        renderNav();
        showHomepage();
    }

    // initial load
    function init() {
        renderNav();
        const current = getCurrentUser();
        if (current) {
            showDashboard();
        } else {
            showHomepage();
        }
    }

    init();
</script>
    <!-- But remove its own <html>, <head>, <body> tags – keep only the inner content from <nav class="navbar"> to </footer> -->
</div>

<style>
    /* Override any portfolio styles that might break the laundromat layout */
    .laundromat-demo .navbar,
    .laundromat-demo .container,
    .laundromat-demo .hero,
    .laundromat-demo .plans,
    .laundromat-demo .dashboard,
    .laundromat-demo footer {
        max-width: 100%;
        margin: 0 auto;
    }
    .laundromat-demo body {
        margin: 0;
        background: #f4f7fc;
    }
    /* Ensure the laundromat footer doesn't clash with your portfolio footer */
    .laundromat-demo footer {
        position: relative;
        background: #0b3b5f;
        color: white;
        border-top: none;
    }
</style>
