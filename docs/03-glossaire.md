# §3 — Glossaire

Document autonome : tout terme technique du DCT SupEvents est défini ici. Les définitions sont contextualisées au projet, pas génériques.

| Terme | Définition | Première occurrence |
|---|---|---|
| **SSO** | Single Sign-On. Authentification unique via le fournisseur d'identité de l'école Sup de Vinci. Un étudiant connecté à son espace école accède directement à SupEvents sans nouveau formulaire. | §4 |
| **OIDC** | OpenID Connect. Protocole d'identité basé sur OAuth 2.0 utilisé par le SSO école pour fournir un `id_token` (identité) et un `access_token` (autorisation) à SupEvents. | §4 |
| **RBAC** | Role-Based Access Control. SupEvents utilise trois rôles : `étudiant`, `organisateur`, `administrateur`. Les rôles sont propagés depuis le SSO via les claims du JWT. | §4 |
| **RGPD** | Règlement Général sur la Protection des Données. Pour SupEvents, encadre la collecte des données étudiantes (nom, email, historique d'inscriptions) et impose un droit à l'oubli sous 30 jours. | §4 |
| **PCI-DSS** | Payment Card Industry Data Security Standard. SupEvents délègue entièrement la conformité à Stripe — aucune donnée bancaire ne transite par nos serveurs. | §4 |
| **Webhook** | Notification HTTPS sortante émise par Stripe vers SupEvents pour confirmer un paiement (`payment_intent.succeeded`, `payment_intent.payment_failed`). Authentifié par signature HMAC `Stripe-Signature`. | §4 |
| **Broker de messages** | Composant intermédiaire (RabbitMQ chez SupEvents) découplant les services producteurs des services consommateurs. Utilisé pour les notifications, les rappels, la génération de QR codes. | §4 |
| **API REST** | Interface HTTPS exposée par le backend SupEvents. Toutes les routes sont préfixées par `/api/v1/`. Format JSON. | §4 |
| **Early-bird** | Type de ticket à tarif réduit, soumis à une limite temporelle ou quantitative définie par l'organisateur. Bascule automatique vers le tarif normal une fois la limite atteinte. | §4 |
| **CRUD** | Create / Read / Update / Delete. Vocabulaire désignant les quatre opérations standards exposées par l'API événements et l'API utilisateurs. | §6.1 |
| **SLA** | Service Level Agreement. Engagement contractuel de disponibilité. SupEvents cible ≥ 99,5 % mensuel. | §9 |
| **P95** | 95e centile d'une distribution de mesures. Pour SupEvents, le SLO de latence cible un p95 < 500 ms sur les routes critiques. | §9 |
| **WCAG** | Web Content Accessibility Guidelines. SupEvents cible la conformité **WCAG 2.1 niveau AA**, vérifiée par audit Axe + Lighthouse. | §9 |
| **i18n** | Internationalisation. SupEvents propose deux langues : français et anglais, sélectionnables par l'utilisateur. | §9 |
| **CI/CD** | Continuous Integration / Continuous Delivery. Pipeline automatique (lint, tests, build, déploiement) exécuté à chaque push sur le dépôt SupEvents. | §9 |
| **Jauge** | Capacité maximale d'inscrits sur un événement. Définie par l'organisateur à la création. Une fois atteinte, l'API refuse toute nouvelle inscription (HTTP 409). | §5 |
| **QR code** | Code-barres bidimensionnel imprimé sur le billet, permettant le contrôle d'accès à l'événement. Généré côté serveur après confirmation du paiement. | §6.2 |
| **Idempotence** | Propriété d'une opération qui produit le même résultat lorsqu'elle est exécutée plusieurs fois avec les mêmes paramètres. Critique pour les retries Stripe et les replays de webhook. | §7 |
| **ADR** | Architecture Decision Record. Document court (format Nygard) capturant **une** décision structurante et son contexte. Stockés dans `/docs/adr/`. | §11 |
| **JWT** | JSON Web Token. Jeton signé contenant l'identité et les rôles de l'utilisateur, transmis dans l'en-tête `Authorization: Bearer <token>` à chaque requête authentifiée. | §6.1 |
