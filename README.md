# 🎯 Simple Shooting Game in Unity

A simple **first-person shooting game** built in Unity.  
This project demonstrates the core gameplay systems of a small FPS:
- Shooting projectiles
- Defeating enemies and a boss
- Managing health and UI
- Unlocking new areas upon enemy defeat

---

## 🧩 Unity Version
- **Unity Version:** 2022.3.17f1 or newer  
- **Render Pipeline:** Built-in  
- **Platform:** PC  
- **Scripting Backend:** IL2CPP (recommended)

---

## 🕹️ Gameplay Overview
- The player shoots rockets to defeat enemies.
- Defeating all enemies unlocks the next area.
- A boss appears after certain conditions are met.
- UI updates dynamically (player HP, enemies left, boss HP).
- Game ends with Game Over or Boss Defeated screens.

---

## 💻 Script Files and Their Purposes

---

### 🧨 `Rocket.cs` — Manages rocket collisions, scoring, and boss hits
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
