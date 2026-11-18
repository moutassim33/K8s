# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :** Moutassim (à confirmer)
**Groupe :** M2 Informatique

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > 3 nœuds : `k3d-lab-server-0` est le control plane (API Server, planification et gestion du cluster), tandis que `k3d-lab-agent-0` et `k3d-lab-agent-1` sont les workers qui exécutent les pods.
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   > `docker ps` affiche quatre conteneurs principaux : `k3d-lab-server-0`, `k3d-lab-agent-0`, `k3d-lab-agent-1` et le load balancer. Dans k3d, chaque nœud Kubernetes est donc concrètement un conteneur Docker exécutant k3s.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   > Un Deployment `web`, son ReplicaSet, un Pod nginx et le Service `web` exposé sur le port 80.
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > Le nom du Pod a changé. Le ReplicaSet a détecté que le nombre de Pods ne correspondait plus à l'état désiré du Deployment et a créé automatiquement un nouveau Pod : boucle de réconciliation.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > Non. La modification effectuée avec `kubectl exec` était uniquement dans le système de fichiers du conteneur. Un nouveau Pod repartait de l'image nginx originale, donc la modification était éphémère.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   > Un `port-forward` vers un Service peut être attaché à un seul Pod. Lorsque ce Pod est supprimé, le tunnel en cours ne reste pas automatiquement connecté au nouveau Pod.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > `create` crée les ressources en mode impératif. `apply` applique un manifeste YAML comme état désiré, peut mettre à jour une ressource existante et permet de conserver l'état dans Git.
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   > Le `selector` choisit les Pods associés au Service grâce aux labels. S'il ne correspond à aucun Pod, le Service n'a aucun endpoint et ne peut acheminer le trafic vers aucun conteneur.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   ```
NAME                    READY   STATUS    RESTARTS   AGE   IP          NODE
site-54dbc4f754-9846v   1/1     Running   0          85s   10.42.1.6   k3d-lab-agent-0
site-54dbc4f754-lrxgm   1/1     Running   0          85s   10.42.0.5   k3d-lab-server-0
site-54dbc4f754-w7qtt   1/1     Running   0          85s   10.42.2.6   k3d-lab-agent-1
   ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    > Le nouveau Pod avait le statut `ImagePullBackOff` (avec également `ErrImagePull` pendant le cycle précédent). `kubectl describe pod -l app=site` a permis de voir l'erreur de téléchargement de l'image.
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    > Oui. Les trois réplicas précédentes restaient disponibles et le Service continuait à les cibler grâce aux endpoints existants. Kubernetes n'a remplacé qu'un Pod à la fois avec une stratégie rolling update.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    > https://shiny-computing-machine-97xqj6g5q5q6275x-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    > Oui. Ils utilisent tous deux l'API Kubernetes du même cluster local et voient le même objet `Deployment/mario` créé dans le namespace `default`.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    > Bonus A réalisé : Ingress `site` sur Traefik, port 8081. Bonus B réalisé : readinessProbe HTTP sur `/`, requêtes et limites CPU/mémoire ajoutées au conteneur.

---

**Score obtenu avec `./scripts/check.sh` :** 18 / 18
