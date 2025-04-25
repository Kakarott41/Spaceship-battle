using UnityEngine;
using UnityEngine.UI;

public class SpaceshipBattleGame : MonoBehaviour
{
    [Header("Movement Settings")]
    public float moveSpeed = 10f;
    public float turnSpeed = 100f;

    [Header("Weapon Settings")]
    public GameObject laserPrefab;
    public Transform firePoint;
    public float laserSpeed = 15f;
    public float fireRate = 0.2f;
    private float nextFireTime = 0f;

    [Header("Health & Shield")]
    public float health = 100f;
    public float shield = 50f;

    [Header("Enemy AI Settings (Only for Enemies)")]
    public Transform target; // The player
    public float enemySpeed = 5f;

    [Header("UI Elements")]
    public Slider healthSlider;
    public Slider shieldSlider;
    public Text ammoText;

    private bool isPlayer = true; // Toggle between player and enemy
    private UIController uiController;

    void Start()
    {
        uiController = FindObjectOfType<UIController>();
    }

    void Update()
    {
        if (isPlayer)
        {
            HandlePlayerMovement();
            HandleShooting();
            UpdateUI();
        }
        else if (target != null)
        {
            HandleEnemyAI();
        }
    }

    // Handle Player Movement
    void HandlePlayerMovement()
    {
        float move = Input.GetAxis("Vertical") * moveSpeed * Time.deltaTime;
        float turn = Input.GetAxis("Horizontal") * turnSpeed * Time.deltaTime;

        transform.Translate(Vector3.forward * move);
        transform.Rotate(Vector3.up * turn);
    }

    // Handle Player Shooting
    void HandleShooting()
    {
        if (Input.GetKey(KeyCode.Space) && Time.time > nextFireTime)
        {
            nextFireTime = Time.time + fireRate;
            ShootLaser();
        }
    }

    // Shooting Laser
    void ShootLaser()
    {
        GameObject laser = Instantiate(laserPrefab, firePoint.position, firePoint.rotation);
        Rigidbody rb = laser.GetComponent<Rigidbody>();
        rb.velocity = firePoint.forward * laserSpeed;
        Destroy(laser, 2f);  // Destroy laser after 2 seconds
    }

    // Handle Enemy AI Behavior (Chase Player)
    void HandleEnemyAI()
    {
        transform.LookAt(target);
        transform.Translate(Vector3.forward * enemySpeed * Time.deltaTime);
    }

    // Take Damage for Health and Shield System
    public void TakeDamage(float damage)
    {
        if (shield > 0)
        {
            shield -= damage;
            if (shield < 0) shield = 0;
        }
        else
        {
            health -= damage;
        }

        if (health <= 0) Die();
    }

    // Handle Player/Enemy Death
    void Die()
    {
        Debug.Log("Player or Enemy Destroyed");
        Destroy(gameObject); // Destroy the spaceship on death
    }

    // Power-Up Pickup Logic
    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("PowerUp"))
        {
            PowerUp powerUp = other.GetComponent<PowerUp>();
            if (powerUp != null)
            {
                if (powerUp.type == PowerUp.PowerType.Health)
                    health += powerUp.value;
                else if (powerUp.type == PowerUp.PowerType.Shield)
                    shield += powerUp.value;

                Destroy(other.gameObject);  // Destroy power-up after pickup
            }
        }
    }

    // Update UI elements (Health & Shield sliders, Ammo counter)
    void UpdateUI()
    {
        if (uiController != null)
        {
            uiController.UpdateUI(health, shield, laserSpeed); // Update the UI values
        }
    }
}

public class PowerUp : MonoBehaviour
{
    public enum PowerType { Health, Shield }
    public PowerType type;
    public float value = 20f; // Amount of health/shield restored

    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            SpaceshipBattleGame playerController = other.GetComponent<SpaceshipBattleGame>();
            if (playerController != null)
            {
                if (type == PowerType.Health)
                    playerController.health += value;
                else if (type == PowerType.Shield)
                    playerController.shield += value;

                Destroy(gameObject);  // Destroy power-up after being picked up
            }
        }
    }
}

public class UIController : MonoBehaviour
{
    public Slider healthSlider;
    public Slider shieldSlider;
    public Text ammoText;

    public void UpdateUI(float health, float shield, float ammo)
    {
        healthSlider.value = health;
        shieldSlider.value = shield;
        ammoText.text = "Ammo: " + Mathf.FloorToInt(ammo);
    }
}
