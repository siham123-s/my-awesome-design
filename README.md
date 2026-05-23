<?php
session_start();
if (!isset($_SESSION['autorise']) || $_SESSION['autorise'] !== "OK") {
    header("Location: conexion.php");
    exit();
}
$is_chef = (isset($_SESSION['role']) && $_SESSION['role'] === 'chef');
$eval_link = $is_chef ? "evaluation_form.php" : "mes_notes.php";
$eval_title = $is_chef ? "Évaluation Personnel" : "Mes Évaluations";
?>
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard | HR Modern</title>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary: #4f46e5;
            --bg-body: #f9fafb;
            /* الخط الجديد */
            --font-main: 'Plus Jakarta Sans', sans-serif;
        }

        * { 
            box-sizing: border-box; 
            margin: 0; 
            padding: 0; 
            font-family: var(--font-main); /* تطبيق الخط على كلشي */
        }

        body {
            background-color: var(--bg-body);
            color: #111827;
            -webkit-font-smoothing: antialiased; /* كيجعل الخط يبان أنعم */
        }

        /* --- Navbar --- */
        .navbar {
            background: rgba(255, 255, 255, 0.8);
            backdrop-filter: blur(10px); /* تأثير الزجاج الضبابي */
            padding: 15px 8%;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 1px solid #f3f4f6;
            position: sticky; top: 0; z-index: 1000;
        }
        .logo { 
            display: flex; 
            align-items: center; 
            gap: 10px; 
            font-weight: 800; 
            color: var(--primary); 
            font-size: 1.2rem;
            letter-spacing: -0.5px;
        }
        .logo-square { background: var(--primary); color: white; padding: 5px 9px; border-radius: 8px; }

        .nav-menu { display: flex; list-style: none; gap: 32px; }
        .nav-menu a { 
            text-decoration: none; 
            color: #4b5563; 
            font-size: 0.9rem; 
            font-weight: 600; 
            transition: 0.3s;
        }
        .nav-menu a:hover, .nav-menu a.active { color: var(--primary); }

        .logout-btn {
            background: #fff1f2; color: #e11d48; padding: 8px 18px; 
            border-radius: 10px; text-decoration: none; font-size: 0.8rem; 
            font-weight: 700; border: 1px solid #ffe4e6; transition: 0.3s;
        }
        .logout-btn:hover { background: #e11d48; color: white; }

        /* --- Slider --- */
        .slider-box { width: 100%; height: 320px; overflow: hidden; position: relative; }
        .slides { display: flex; width: 300%; height: 100%; animation: slideAnim 12s infinite ease-in-out; }
        .slide { 
            width: 100%; height: 100%; background-size: cover; background-position: center;
            display: flex; align-items: center; justify-content: center; position: relative;
        }
        .slide::after { content: ''; position: absolute; inset: 0; background: rgba(0,0,0,0.35); }
        .slide h2 { 
            color: white; 
            position: relative; 
            z-index: 1; 
            font-size: 2.5rem; 
            font-weight: 800; 
            letter-spacing: -1px;
        }

        @keyframes slideAnim {
            0%, 25% { transform: translateX(0); }
            33%, 58% { transform: translateX(-33.33%); }
            66%, 91% { transform: translateX(-66.66%); }
            100% { transform: translateX(0); }
        }

        /* --- Main Content --- */
        .main-container { max-width: 1000px; margin: 60px auto; padding: 0 20px; }
        
        .header-section { text-align: center; margin-bottom: 60px; }
        .header-section h1 { 
            font-size: 2.6rem; 
            font-weight: 800; 
            letter-spacing: -1.5px; 
            color: #111827;
            margin-bottom: 10px;
        }
        .header-section p { color: #6b7280; font-size: 1.1rem; font-weight: 500; }

        /* --- Grid 2x2 --- */
        .grid-cards { display: grid; grid-template-columns: 1fr 1fr; gap: 32px; }

        /* --- Card Style (Plus Jakarta Style) --- */
        .card {
            background: white; 
            padding: 45px; 
            border-radius: 45px; /* زوايا دائرية أكثر نعومة */
            text-decoration: none; 
            color: inherit;
            border: 1px solid #f3f4f6; 
            transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275); 
            display: flex; 
            flex-direction: column;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.02), 0 2px 4px -1px rgba(0, 0, 0, 0.02);
        }
        .card:hover { 
            transform: translateY(-8px) scale(1.01); 
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.05), 0 10px 10px -5px rgba(0, 0, 0, 0.02);
            border-color: #e5e7eb;
        }

        .icon-box {
            width: 58px; height: 58px; border-radius: 16px;
            display: flex; align-items: center; justify-content: center; margin-bottom: 28px;
        }
        .icon-box svg { width: 26px; height: 26px; stroke-width: 2.2; fill: none; stroke: white; }

        .card h3 { 
            font-size: 1.35rem; 
            font-weight: 700; 
            margin-bottom: 14px; 
            letter-spacing: -0.4px;
        }
        .card p { 
            font-size: 0.95rem; 
            color: #6b7280; 
            line-height: 1.7; 
            font-weight: 400;
            flex-grow: 1; 
            margin-bottom: 30px; 
        }
        .card-link { 
            font-size: 0.85rem; 
            font-weight: 800; 
            display: flex; 
            align-items: center; 
            gap: 6px; 
            letter-spacing: 0.5px;
        }

        /* Colors */
        .c-blue .icon-box { background: linear-gradient(135deg, #3b82f6, #2563eb); } 
        .c-blue .card-link { color: #2563eb; }

        .c-red .icon-box { background: linear-gradient(135deg, #f87171, #ef4444); } 
        .c-red .card-link { color: #ef4444; }

        .c-green .icon-box { background: linear-gradient(135deg, #34d399, #10b981); } 
        .c-green .card-link { color: #10b981; }

        .c-indigo .icon-box { background: linear-gradient(135deg, #818cf8, #6366f1); } 
        .c-indigo .card-link { color: #6366f1; }

        @media (max-width: 768px) { 
            .grid-cards { grid-template-columns: 1fr; } 
            .header-section h1 { font-size: 2rem; }
        }
    </style>
</head>
<body>

    <nav class="navbar">
        <div class="logo"><div class="logo-square">GP</div> GestionPersonnel</div>
        
        <ul class="nav-menu">
            <li><a href="#" class="active">Accueil</a></li>
            <li><a href="profileee.php">Mon Profil</a></li>
            <li><a href="consignes.php">Consignes</a></li>
            <li><a href="contact.php">contacte</a></li>
        </ul>

        <div style="display: flex; align-items: center; gap: 20px;">
            <span style="font-size: 0.85rem; color: #4b5563; font-weight: 500;">Bonjour, <b><?php echo $_SESSION['nom']; ?></b></span>
            <a href="logout.php" class="logout-btn">Déconnexion</a>
        </div>
    </nav>

    <div class="slider-box">
        <div class="slides">
            <div class="slide" style="background-image: url('https://images.unsplash.com/photo-1497215728101-856f4ea42174?auto=format&fit=crop&w=1200&q=80')"><h2>Espace Collaborateur</h2></div>
            <div class="slide" style="background-image: url('https://images.unsplash.com/photo-1486406146926-c627a92ad1ab?auto=format&fit=crop&w=1200&q=80')"><h2>Innovation & RH</h2></div>
            <div class="slide" style="background-image: url('https://images.unsplash.com/photo-1557804506-669a67965ba0?auto=format&fit=crop&w=1200&q=80')"><h2>Votre avenir ici</h2></div>
        </div>
    </div>

    <main class="main-container">
        <div class="header-section">
            <h1>ESPACE FONCTIONNAIRE</h1>
            <p>Bonjour ! Que souhaitez-vous faire aujourd'hui ?</p>
        </div>

        <div class="grid-cards">
            <a href="congee.php" class="card c-blue">
                <div class="icon-box">
                    <svg viewBox="0 0 24 24" stroke="currentColor"><rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect><line x1="16" y1="2" x2="16" y2="6"></line><line x1="8" y1="2" x2="8" y2="6"></line><line x1="3" y1="10" x2="21" y2="10"></line></svg>
                </div>
                <h3>Demande de Congé</h3>
                <p>Soumettez vos demandes de repos annuel ou exceptionnel en quelques clics rapides.</p>
                <div class="card-link">Accéder au portail →</div>
            </a>

            <a href="AbsenceMaladie.php" class="card c-red">
                <div class="icon-box">
                    <svg viewBox="0 0 24 24" stroke="currentColor"><path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l8.72-8.72 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path></svg>
                </div>
                <h3>Absence Maladie</h3>
                <p>Déclarez un arrêt de travail et téléchargez votre certificat médical en toute sécurité.</p>
                <div class="card-link">Faire une déclaration →</div>
            </a>

            <a href="scaaaan.php" class="card c-green">
                <div class="icon-box">
                    <svg viewBox="0 0 24 24" stroke="currentColor"><polyline points="23 4 23 10 17 10"></polyline><polyline points="1 20 1 14 7 14"></polyline><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"></path></svg>
                </div>
                <h3>Pointage Présence</h3>
                <p>Scannez votre code QR pour enregistrer votre entrée ou sortie du jour instantanément.</p>
                <div class="card-link">Scanner maintenant →</div>
            </a>

            <a href="<?php echo $eval_link; ?>" class="card c-indigo">
                <div class="icon-box">
                    <svg viewBox="0 0 24 24" stroke="currentColor"><polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon></svg>
                </div>
                <h3><?php echo $eval_title; ?></h3>
                <p>Consultez vos notes annuelles و les appréciations de votre chef de service en un clic.</p>
                <div class="card-link">Voir mon bilan →</div>
            </a>
        </div>
    </main>

</body>
</html>
