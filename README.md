# Dockyard Chaos in Unity

![Dockyard Gif](/images/Dockyard.gif)

A simple **first-person shooting game** built in Unity.  
This project demonstrates the core FPS gameplay systems — shooting, enemy waves, boss fight mechanics, health and UI management, and destructible objects.

---

##  Gameplay Overview
- The player shoots **rockets** to defeat enemies **or destroy objects**.
- The player needs to survive and avoid projectiles from enemies.
- Destroying all enemies unlocks the next area.
- A final **boss** appears after certain conditions are met.
- The **UI updates dynamically** — showing player HP, enemies left, and boss HP.
- The game ends with either a **Game Over** or **Boss Defeated** screen.

---

## Notes / What’s Not Included
- This project focuses on **shooting mechanics, enemy interactions, destructible objects, boss logic, and UI updates**.
- **Player movement, scene transitions, or other basic FPS mechanics** are not included.
- All scripts are provided to showcase core gameplay logic; additional systems would need to be implemented separately.

---

## Script Files and Their Purposes

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

### `EnemyShootScript.cs` — Controls enemy shooting behavior

```csharp
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

---

### `PlayerHealth.cs` — Handles player health, death, and Game Over screen

```csharp
using UnityEngine;

public class PlayerHealth : MonoBehaviour
{
    public int maxHealth = 10;
    public int currentHealth;
    public GameObject gameOverScreen;

    private void Start()
    {
        currentHealth = maxHealth;
        gameOverScreen.SetActive(false);
    }

    public void TakeDamage(int damage)
    {
        currentHealth -= damage;

        if (currentHealth <= 0)
        {
            Die();
        }
    }

    void Die()
    {
        gameOverScreen.SetActive(true);
        Cursor.lockState = CursorLockMode.None;
        Cursor.visible = true;
        Time.timeScale = 0f;
    }
}
```

---

### `HpEnemeyText.cs` — Updates UI text (HP, enemies left, boss HP)

```csharp
using UnityEngine;
using TMPro;

public class HpEnemeyText : MonoBehaviour
{
    public TMP_Text hpText;
    public TMP_Text enemiesLeftText;
    public TMP_Text bossText;
    public TMP_Text bossHpText;

    private PlayerHealth playerHealth;
    private EnemyCounter enemyCounter;
    public Rocket rocketScript;
    public GameObject bossDefeatedObject;

    void Start()
    {
        playerHealth = FindObjectOfType<PlayerHealth>();
        enemyCounter = FindObjectOfType<EnemyCounter>();
        UpdateHealthText();
        bossText.gameObject.SetActive(false);
        bossHpText.gameObject.SetActive(false);
    }

    void Update()
    {
        UpdateHealthText();
        CheckForBoss();

        if (Rocket.bossHitCount == 0)
            UpdateEnemiesLeftText();
        else
            UpdateBossHpText();
    }

    void UpdateHealthText()
    {
        if (playerHealth != null)
            hpText.text = "HP: " + playerHealth.currentHealth.ToString();
    }

    void UpdateEnemiesLeftText()
    {
        if (enemyCounter != null)
        {
            if (enemyCounter.totalEnemies > 0)
                enemiesLeftText.text = "Enemies Left: " + enemyCounter.totalEnemies.ToString();
            else
                enemiesLeftText.text = "No More Enemies";
        }
    }

    void CheckForBoss()
    {
        GameObject boss = GameObject.FindGameObjectWithTag("Boss");

        if (boss != null)
        {
            enemiesLeftText.gameObject.SetActive(false);
            bossText.gameObject.SetActive(true);
            bossHpText.gameObject.SetActive(true);
            bossText.text = "Boss Arrived! Destroy 'Em!";
            UpdateBossHpText();
        }
        else
        {
            enemiesLeftText.gameObject.SetActive(true);
            bossText.gameObject.SetActive(false);
            bossHpText.gameObject.SetActive(false);
        }
    }

    void UpdateBossHpText()
    {
        int remainingHits = rocketScript.bossHitLimit - Rocket.bossHitCount;

        if (remainingHits > 0)
        {
            bossHpText.text = "Boss HP: " + remainingHits;
        }
        else
        {
            bossHpText.text = "Boss Defeated!";

            if (bossDefeatedObject != null)
            {
                bossDefeatedObject.SetActive(true);
                Cursor.lockState = CursorLockMode.None;
                Cursor.visible = true;
                Time.timeScale = 0f;
            }
        }
    }
}

```

---

### `BulletScript.cs` — Handles bullets fired by enemies

```csharp
using UnityEngine;

public class BulletScript : MonoBehaviour
{
    public float speed = 10f;
    public int damage = 1;

    void Update()
    {
        transform.Translate(Vector3.forward * speed * Time.deltaTime);
    }

    void OnCollisionEnter(Collision collision)
    {
        if (collision.gameObject.CompareTag("Player"))
        {
            collision.gameObject.GetComponent<PlayerHealth>().TakeDamage(damage);
            Destroy(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }
}
```

