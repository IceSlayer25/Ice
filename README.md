# Ice

public class CharacterShooting : MonoBehaviour {
    public Gun gun;

    public int shootButton;
    public KeyCode reloadKey;

    void Update() {
        if(Input.GetMouseButton(shootButton)) {
            gun.Shoot();
        }

        if(Input.GetKeyDown(reloadKey)) {
            gun.Reload();
        }
    }
}
Gun.cs
using UnityEngine;

public class Gun : MonoBehaviour {
    public enum ShootState {
        Ready,
        Shooting,
        Reloading
    }

    // How far forward the muzzle is from the centre of the gun
    private float muzzleOffset;

    [Header("Magazine")]
    public GameObject round;
    public int ammunition;

    [Range(0.5f, 10)] public float reloadTime;

    private int remainingAmmunition;

    [Header("Shooting")]
    // How many shots the gun can make per second
    [Range(0.25f, 25)] public float fireRate;

    // The number of rounds fired each shot
    public int roundsPerShot;

    [Range(0.5f, 100)] public float roundSpeed;

    // The maximum angle that the bullet's direction can vary,
    // in both the horizontal and vertical axes
    [Range(0, 45)] public float maxRoundVariation;

    private ShootState shootState = ShootState.Ready;

    // The next time that the gun is able to shoot at
    private float nextShootTime = 0;

    void Start() {
        muzzleOffset = GetComponent<Renderer>().bounds.extents.z;
        remainingAmmunition = ammunition;
    }

    void Update() {
        switch(shootState) {
            case ShootState.Shooting:
                // If the gun is ready to shoot again...
                if(Time.time > nextShootTime) {
                    shootState = ShootState.Ready;
                }
                break;
            case ShootState.Reloading:
                // If the gun has finished reloading...
                if(Time.time > nextShootTime) {
                    remainingAmmunition = ammunition;
                    shootState = ShootState.Ready;
                }
                break;
        }
    }

    /// Attempts to fire the gun
    public void Shoot() {
        // Checks that the gun is ready to shoot
        if(shootState == ShootState.Ready) {
            for(int i = 0; i < roundsPerShot; i++) {
                // Instantiates the round at the muzzle position
                GameObject spawnedRound = Instantiate(
                    round,
                    transform.position + transform.forward * muzzleOffset,
                    transform.rotation
                );

                // Add a random variation to the round's direction
                spawnedRound.transform.Rotate(new Vector3(
                    Random.Range(-1f, 1f) * maxRoundVariation,
                    Random.Range(-1f, 1f) * maxRoundVariation,
                    0
                ));

                Rigidbody rb = spawnedRound.GetComponent<Rigidbody>();
                rb.velocity = spawnedRound.transform.forward * roundSpeed;
            }

            remainingAmmunition--;
            if(remainingAmmunition > 0) {
                nextShootTime = Time.time + (1 / fireRate);
                shootState = ShootState.Shooting;
            } else {
                Reload();
            }
        }
    }

    /// Attempts to reload the gun
    public void Reload() {
        // Checks that the gun is ready to be reloaded
        if(shootState == ShootState.Ready) {
            nextShootTime = Time.time + reloadTime;
            shootState = ShootState.Reloading;
        }
    }
}
Round.cs
using UnityEngine;

public class Round : MonoBehaviour {
    public float damage;

    void OnCollisionEnter(Collision other) {
        Target target = other.gameObject.GetComponent<Target>();
        // Only attempts to inflict damage if the other game object has
        // the 'Target' component
        if(target != null) {
            target.Hit(damage);
            Destroy(gameObject); // Deletes the round
        }
    }
}
Target.cs
using UnityEngine;

public class Target : MonoBehaviour {
    public float health;

    void Update() {
        if(health <= 0) {
            Destroy(gameObject);
        }
    }

    /// 'Hits' the target for a certain amount of damage
    public void Hit(float damage) {
        health -= damage;
    }
}