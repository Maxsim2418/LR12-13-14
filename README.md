# LR12-13-14[LR12-13-14.zip](https://github.com/Maxsim2418/LR12-13-14/files/10146007/LR12-13-14.zip)

Борнеман М.А

ЭВТ-70

Игровой движок:Unity

Лабораторная работа №12,13,14

Тема: SpaceShooter

Цель: приобрести навыки в разработке игрового проекта SpaceShooter

Ход работы:

1.	Выполнение работы

1.	Настройка сцены, иерархии, использованные объекты и скрипты.
 
![image](https://user-images.githubusercontent.com/119674602/205432608-560903b1-ebaa-44ba-b684-0bba80ccc288.png)

Рисунок 12.1 – Постройка сцены.

2.	Была реализована функция жизней и смертей, корректировка слоёв между объектами, бессмертие в течение короткого времени.

Листинг 12.1 DamageHandler.cs

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DamageHandler : MonoBehaviour
{
    public int health = 1;
    public float invulnPeriod = 0;
    float invulnTimer = 0;
    int correctLayer;
    SpriteRenderer spriteRend;
    void Start()
    {
        correctLayer = gameObject.layer;

        spriteRend = GetComponent<SpriteRenderer>();

        if(spriteRend == null)
        {
            spriteRend = transform.GetComponentInChildren<SpriteRenderer>();

            if(spriteRend == null)
            {
                Debug.LogError("Object ' " + gameObject.name + "' has no sprite render.");
            }
            
        }
    }
    private void OnTriggerEnter2D()
    {
            health--;
            invulnTimer = invulnPeriod;
            gameObject.layer = 10;
    }
    void Update()
    {
        if(invulnTimer >0)
        {
        invulnTimer -= Time.deltaTime;
            if (invulnTimer <= 0)
            {
                gameObject.layer = correctLayer;
                if(spriteRend != null)
                {
                    spriteRend.enabled = true;
                }
                
            }
            else
            {
                if (spriteRend != null)
                {
                    spriteRend.enabled = !spriteRend.enabled;
                }
            }
        }
        if (health <= 0)
        {
            Die();
        }
    }
    void Die()
    {
        Destroy(gameObject);
    }
}

3.	Реализована стрельба противников.

Листинг 12.2 EnemyShooting.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyShooting : MonoBehaviour
{
    public Vector3 bulletOffset = new Vector3(0, 0.5f, 0);
    public GameObject bulletPrefab;
    int bulletLayer;
    public float fireDelay = 0.25f;
    float cooldownTimer = 0;
    Transform player;
    void Start()
    {
        bulletLayer = gameObject.layer;
    }
    void Update()
    {
        if (player == null)
        {
            GameObject go = GameObject.FindWithTag("Player");

            if (go != null)
            {
                player = go.transform;
            }
        }
        cooldownTimer -= Time.deltaTime;
        if (cooldownTimer <= 0 && player != null && Vector3.Distance(transform.position, player.position) < 4)
        {
            cooldownTimer = fireDelay;
            Vector3 offset = transform.rotation * bulletOffset;
            GameObject bulletGO = (GameObject)Instantiate(bulletPrefab, transform.position + offset, transform.rotation);
            bulletGO.layer = bulletLayer;
        }
    }
}

4.	Реализован респавн противников в определённом радиусе и промежутке времени.

Листинг 12.3 EnemySpawner.cs

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemySpawner : MonoBehaviour
{
    public GameObject enemyPrefab;
    float spawnDistance = 12f;
    float enemyRate = 5;
    float nextEnemy = 1;
    void Update()
    {
        nextEnemy -= Time.deltaTime; 
        if(nextEnemy <= 0)
        {
            nextEnemy = enemyRate;
            enemyRate *= 0.9f;
            if (enemyRate < 2)
                enemyRate = 2;
            Vector3 offset = Random.onUnitSphere;
            offset.z = 0;
            offset = offset.normalized * spawnDistance;
            Instantiate(enemyPrefab, transform.position + offset, Quaternion.identity);
        }
    }
}

5.	Реализована скорость поворота игрока а так же направление выстрелов.

Листинг 12.4 FacesPlayer.cs

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class FacesPlayer : MonoBehaviour
{
    public float rotSpeed = 90f;
    Transform player;
    void Update()
    {
        if(player == null)
        {
            GameObject go = GameObject.FindWithTag ("Player");
            if(go != null)
            {
                player = go.transform; 
            }
        }
        if (player == null)
            return;
        Vector3 dir = player.position - transform.position;
        dir.Normalize();
        float zAngle = Mathf.Atan2(dir.y, dir.x) * Mathf.Rad2Deg - 90;
        Quaternion desiredRot = Quaternion.Euler(0, 0, zAngle);
        transform.rotation = Quaternion.RotateTowards(transform.rotation, desiredRot, rotSpeed * Time.deltaTime);
        
    }
}

6.	Настроена скорость перемещения игрока вперёд.

Листинг 12.5 MoveForward.cs

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MoveForward : MonoBehaviour
{
    public float maxSpeed = 5f;
    void Update()
    {
        Vector3 pos = transform.position;
        Vector3 velocity = new Vector3(0,  maxSpeed * Time.deltaTime, 0);
        pos += transform.rotation * velocity;
        transform.position = pos;
    }
}

7.	Сделано перемещение игрока.

Листинг 12.6 PlayerMovement.cs

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerMovement : MonoBehaviour
{
    public float maxSpeed = 5f;
    public float rotSpeed = 180f;
    float shipBoundaryRadius = 0.5f;
    void Update()
    {
        Quaternion rot = transform.rotation;
        float z = rot.eulerAngles.z;
        z -= Input.GetAxis("Horizontal") * rotSpeed * Time.deltaTime;
        rot = Quaternion.Euler( 0, 0, z );
        transform.rotation = rot;
        Vector3 pos = transform.position;
        Vector3 velocity = new Vector3(0, Input.GetAxis("Vertical") * maxSpeed * Time.deltaTime, 0);
        pos += rot * velocity;
        if(pos.y+shipBoundaryRadius > Camera.main.orthographicSize)
        {
            pos.y = Camera.main.orthographicSize - shipBoundaryRadius;
        }
        if (pos.y - shipBoundaryRadius < -Camera.main.orthographicSize)
        {
            pos.y = -Camera.main.orthographicSize + shipBoundaryRadius;
        }
        float screenRatio = (float)Screen.width / (float)Screen.height;
        float widthOrtho = Camera.main.orthographicSize * screenRatio;
        if (pos.x + shipBoundaryRadius > widthOrtho)
        {
            pos.x = widthOrtho - shipBoundaryRadius;
        }
        if (pos.x - shipBoundaryRadius < -widthOrtho)
        {
            pos.x = -widthOrtho + shipBoundaryRadius;
        }
        transform.position = pos;
    }
}

8.	Была сделана возможность стрельбы для игрока

Листинг 12.7 PlayerShooting.cs

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerShooting : MonoBehaviour
{
    public Vector3 bulletOffset = new Vector3(0, 0.5f, 0);
    public GameObject bulletPrefab;
    int bulletLayer;
    public float fireDelay = 0.25f;
    float cooldownTimer = 0;
    void Start()
    {
        bulletLayer = gameObject.layer;
    }
    void Update()
    {
        cooldownTimer -= Time.deltaTime;
        if (Input.GetButton("Fire1") && cooldownTimer <= 0)
        {
            cooldownTimer = fireDelay;
            Vector3 offset = transform.rotation * bulletOffset;
            GameObject bulletGO = (GameObject)Instantiate(bulletPrefab, transform.position + offset, transform.rotation);
            bulletGO.layer = gameObject.layer;
        }
    }
}

9.	Реализован респавн игрока и экран поражения при утрате всех жизней.

Листинг 12.8 PlayerSpawner.cs

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerSpawner : MonoBehaviour
{
    public GameObject playerPrefab;
    GameObject playerInstance;
    public int numLives = 4;
    float respawnTimer;
    void Start()
    {
        SpawnPLayer();
    }
    void SpawnPLayer()
    {
        numLives--;
        respawnTimer = 1;
        playerInstance = (GameObject)Instantiate(playerPrefab, transform.position, Quaternion.identity);

    }
    void Update()
    {
        if (playerInstance == null && numLives > 0)
        {
            respawnTimer -= Time.deltaTime;
            if (respawnTimer <= 0)
            {
                SpawnPLayer();
            }
        }
    }
    void OnGUI()
    {
        if (numLives > 0 || playerInstance!= null)
        {
            GUI.Label(new Rect(0, 0, 100, 50), "Lives Left: " + numLives);
        }
        else
        {
            GUI.Label(new Rect(Screen.width / 2 - 50, Screen.height/2 - 25, 100, 50), "Game Over, Man!");
        }
    }
}

10.	Был написан скрипт самоуничтожения пуль игрока и противников через определённое время.

Листинг 12.9 SelfDestruct.cs

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class SelfDestruct : MonoBehaviour
{
    public float timer = 1f;
    void Update()
    {
        timer -= Time.deltaTime;
        if (timer <= 0)
        {
            Destroy(gameObject);
        }
    }
}

2.	Вывод

В ходе проделанной работы была сделана игра SpaceShooter.
