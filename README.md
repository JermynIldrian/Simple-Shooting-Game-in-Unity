# 🎯 Simple Shooting Game in Unity

A simple **first-person shooting game** built in Unity.  
This project demonstrates the core FPS gameplay systems — shooting, enemy waves, boss fight mechanics, health and UI management, and destructible objects.

---

## 🕹️ Gameplay Overview
- The player shoots **rockets** to defeat enemies **or destroy objects**.
- Destroying all enemies unlocks the next area.
- A final **boss** appears after certain conditions are met.
- The **UI updates dynamically** — showing player HP, enemies left, and boss HP.
- The game ends with either a **Game Over** or **Boss Defeated** screen.

---

## 💻 Script Files and Their Purposes

---

### `Rocket.cs` — Manages rocket collisions, scoring, and boss hits
```csharp
using UnityEngine;

public class Rocket : MonoBehaviour
{
    public EnemyCounter enemyCounter;
    public static int bossHitCount = 0;
    public int bossHitLimit = 20;

    private void OnCollisionEnter(Collision collision)
    {
        if (collision.gameObject.CompareTag("Unbreakable"))
        {
            Destroy(gameObject);
            return;
        }

        if (collision.gameObject.CompareTag("Object"))
        {
            Destroy(collision.gameObject);
            Destroy(gameObject);
            UI.AddScore(5);
            return;
        }

        if (collision.gameObject.CompareTag("Enemy"))
        {
            if (enemyCounter != null)
                enemyCounter.DecreaseEnemyCount();

            Destroy(collision.gameObject);
            Destroy(gameObject);
            UI.AddScore(10);
            return;
        }

        if (collision.gameObject.CompareTag("Boss"))
        {
            bossHitCount++;
            if (bossHitCount >= bossHitLimit)
            {
                Destroy(collision.gameObject);
                if (enemyCounter != null)
                    enemyCounter.DecreaseEnemyCount();

                UI.AddScore(50);
                Debug.Log("Boss destroyed! Total hits: " + bossHitCount);
            }

            Destroy(gameObject);
        }
    }
}

using UnityEngine;

public class EnemyShootScript : MonoBehaviour
{
    public Transform player;
    public float shootingRange = 10f;
    public float fireRate = 1f;
    public GameObject bulletPrefab;
    public Transform shootPoint;

    private float nextFireTime = 0f;

    void Update()
    {
        float distanceToPlayer = Vector3.Distance(transform.position, player.position);

        if (distanceToPlayer <= shootingRange && Time.time >= nextFireTime)
        {
            Shoot();
        }
    }

    void Shoot()
    {
        Vector3 directionToPlayer = (player.position - shootPoint.position).normalized;
        GameObject bullet = Instantiate(bulletPrefab, shootPoint.position, Quaternion.LookRotation(directionToPlayer));
        Rigidbody bulletRb = bullet.GetComponent<Rigidbody>();

        if (bulletRb != null)
        {
            bulletRb.linearVelocity = directionToPlayer * bulletRb.linearVelocity.magnitude;
        }

        nextFireTime = Time.time + 1f / fireRate;
    }
}
```

---

### `EnemyCounter.cs` — Tracks total enemies and unlocks new areas

```csharp
using UnityEngine;

public class EnemyCounter : MonoBehaviour
{
    public int totalEnemies;
    public GameObject nextArea;

    void Start()
    {
        GameObject[] enemies = GameObject.FindGameObjectsWithTag("Enemy");
        totalEnemies = enemies.Length;
        Debug.Log("Total Enemies: " + totalEnemies);
    }

    public void DecreaseEnemyCount()
    {
        totalEnemies--;
        Debug.Log("Enemies Remaining: " + totalEnemies);

        if (totalEnemies <= 0 && nextArea != null)
        {
            nextArea.SetActive(true);
            Debug.Log("All enemies are dead! The next area is now unlocked.");
        }
    }
}
```
