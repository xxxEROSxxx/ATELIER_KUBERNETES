# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :** RIGUIDEL Eros-Nathan
**Groupe :** L3 SRD Cnam - MONT 26-27

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > 3 nœuds. 1 control-plane (le chef) : il dirige le cluster et décide où placer les pods. 2 workers : ce sont eux qui font tourner les pods.
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   > On voit un conteneur par nœud (server-0, agent-0, agent-1) plus le load balancer. Dans k3d, un nœud = un simple conteneur Docker qui fait tourner Kubernetes (k3s).

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   > Un Deployment, un ReplicaSet et un Pod. Le Service web a été créé ensuite par `kubectl expose`.
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > Un nouveau pod est apparu tout seul, avec un autre nom. Le ReplicaSet surveille en permanence : s'il manque un pod, il en recrée un pour revenir au nombre demandé.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > Non. Un conteneur est éphémère : le nouveau pod repart de l'image nginx d'origine, donc la modification faite à la main est perdue.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   > Le port-forward est branché sur un seul pod précis. Quand ce pod est supprimé, le tunnel n'a plus de destination et il casse.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > `create` = on donne un ordre à la main, une seule fois (erreur si ça existe déjà). `apply` = on décrit ce qu'on veut dans un fichier YAML, et Kubernetes crée ou met à jour pour que le cluster corresponde au fichier.
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   > Il dit au Service à quels pods envoyer le trafic, grâce à leurs labels (ex : app=site). S'il ne trouve aucun pod, le Service n'envoie le trafic nulle part : le site ne répond pas.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   > Les 3 pods sont répartis sur 3 nœuds différents : server-0, agent-0 et agent-1.
   ```
   NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE               NOMINATED NODE   READINESS GATES
   site-7b7586ddb4-26xvm   1/1     Running   0          9s      10.42.0.7    k3d-lab-server-0   <none>           <none>
   site-7b7586ddb4-7pcjq   1/1     Running   0          9s      10.42.1.8    k3d-lab-agent-1    <none>           <none>
   site-7b7586ddb4-sffxj   1/1     Running   0          2m20s   10.42.2.10   k3d-lab-agent-0    <none>           <none>
   ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    > Statut ErrImagePull puis ImagePullBackOff : l'image nginx:9.99 n'existe pas. La cause est dans les Events de `kubectl describe pod -l app=site`.
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    > Oui. Kubernetes remplace les pods un par un et ne supprime un ancien pod que quand le nouveau est prêt. Le nouveau n'a jamais démarré, donc les anciens pods ont continué à servir le site.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    > https://stunning-space-guide-v69944w64p7whwjg9-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    > Oui, `kubectl get deploy mario` le retrouve bien. Portainer et kubectl parlent tous les deux à l'API Server, sur le control-plane. Portainer est juste une interface graphique au-dessus.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    > A : Ingress Traefik, le site répond sur le port 8081 même si on supprime des pods. B : readinessProbe sur / (le trafic ne va qu'aux pods prêts) + limites CPU et mémoire.

---

**Score obtenu avec `./scripts/check.sh` :** 18 / 18