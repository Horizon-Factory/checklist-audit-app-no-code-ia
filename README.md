# Checklist d'audit pour une application no-code ou créée avec l'IA

25 points pour savoir si une app construite avec Bubble, Lovable, Claude, Cursor ou un outil similaire peut encaisser sa croissance. Chaque point se vérifie en quelques minutes, avec une commande quand c'est possible.

Cette checklist accompagne l'article [Les 5 limites des applications créées en no-code ou avec l'IA](https://horizonfactory.fr/fr/blog/limites-applications-no-code-ia).

> Lancez ces vérifications uniquement sur une application qui vous appartient. Faites les tests de charge sur une copie, jamais en production.

## Comment l'utiliser

Comptez un point par case cochée. Une réponse « je ne sais pas » vaut non.

| Score | Lecture |
| --- | --- |
| 21 à 25 | Bases saines. Corrigez les points restants au fil de l'eau. |
| 13 à 20 | L'app tourne, mais un pic de trafic ou une mauvaise mise à jour peut la faire tomber. |
| 0 à 12 | Commencez par les sauvegardes et la sécurité, avant toute nouvelle fonctionnalité. |

Quel que soit le score, un point **critique** non coché passe en priorité.

## 1. Sécurité et accès

- [ ] **Critique.** Chaque table exposée a le Row Level Security activé (Supabase, Postgres).

  ```sql
  -- Doit renvoyer zéro ligne
  SELECT tablename FROM pg_tables
  WHERE schemaname = 'public' AND NOT rowsecurity;
  ```

- [ ] **Critique.** Aucune règle d'accès n'ouvre une table à tout le monde.

  ```sql
  -- Chaque ligne renvoyée doit être une donnée réellement publique
  SELECT tablename, policyname, cmd FROM pg_policies
  WHERE schemaname = 'public' AND (qual = 'true' OR with_check = 'true');
  ```

- [ ] **Critique.** Aucun secret dans le code envoyé au navigateur ni dans l'historique Git.

  ```bash
  gitleaks git .       # historique Git
  gitleaks dir dist    # build (adaptez le dossier : dist, build, .next)
  ```

  La clé `service_role` de Supabase ne doit jamais apparaître côté client.

- [ ] Les droits sont vérifiés côté serveur. Masquer un bouton dans l'interface ne protège rien.
- [ ] L'inscription limite les abus : e-mail vérifié, limite de tentatives, protection contre les robots.
- [ ] Les dépendances n'ont aucune faille critique connue.

  ```bash
  npm audit --omit=dev --audit-level=critical
  ```

## 2. Montée en charge

- [ ] L'app a été testée sur une copie avec plus de trafic que votre pic actuel.

  ```bash
  npx autocannon -c 50 -d 30 https://votre-copie-de-test.example
  ```

- [ ] Les tables les plus lues utilisent des index.

  ```sql
  -- Un seq_scan élevé sur une grosse table signale souvent un index manquant
  SELECT relname, seq_scan, idx_scan, n_live_tup
  FROM pg_stat_user_tables
  ORDER BY seq_tup_read DESC LIMIT 10;
  ```

- [ ] Les listes sont paginées. Aucun écran ne charge une table entière.
- [ ] Vous êtes alerté d'une panne avant vos clients (sonde de disponibilité, suivi des erreurs).
- [ ] L'hébergement est dimensionné à partir de mesures réelles.

## 3. Mises à jour sans risque

- [ ] Un environnement de test séparé existe, avec sa propre base de données.
- [ ] Le code est versionné, et chaque mise en ligne correspond à une version identifiable.
- [ ] Vous savez revenir à la version précédente en quelques minutes.
- [ ] Les parcours vitaux (inscription, connexion, paiement) sont testés avant chaque mise en ligne.
- [ ] **Critique.** Des sauvegardes automatiques tournent chaque jour, hors de la plateforme principale.
- [ ] **Critique.** Une sauvegarde a déjà été restaurée pour vérifier qu'elle fonctionne.

  ```bash
  pg_dump -Fc "$SOURCE_DATABASE_URL" -f sauvegarde.dump
  pg_restore --no-owner -d "$TEST_DATABASE_URL" sauvegarde.dump
  ```

## 4. Coûts

- [ ] Vous suivez chaque mois le coût de l'app par utilisateur actif.
- [ ] Les coûts montent moins vite que le nombre d'utilisateurs.
- [ ] Des alertes de budget sont actives sur chaque service facturé à l'usage.

## 5. Évolutivité et propriété

- [ ] Quelqu'un dans l'équipe sait expliquer comment l'app fonctionne, sans relire tout le code.
- [ ] Les fonctionnalités demandées par vos clients sont réalisables avec votre outil actuel.
- [ ] **Critique.** Le code, l'hébergement et le nom de domaine sont au nom de votre société.
- [ ] Si la plateforme fermait demain, vous pourriez récupérer l'application, pas seulement les données.
- [ ] Vous savez où sont hébergées vos données, et ce choix est conforme au RGPD.

## Contribuer

Un point manque ou une commande ne marche pas sur votre stack ? Ouvrez une issue ou une pull request.

## Licence

[MIT](LICENSE), par [Horizon Factory](https://horizonfactory.fr), agence de développement sur-mesure.
