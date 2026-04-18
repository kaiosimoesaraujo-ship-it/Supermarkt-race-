using UnityEngine;
using System.Collections.Generic;

public class SupermarketRace : MonoBehaviour
{
    public static SupermarketRace instance;

    [Header("Players")]
    public List<PlayerData> players = new List<PlayerData>();

    [Header("Podium")]
    public Transform podium1;
    public Transform podium2;
    public Transform podium3;

    void Awake()
    {
        instance = this;
    }

    void Start()
    {
        Debug.Log("🏁 Corrida iniciada!");
    }

    // ================= PLAYER =================
    public void RegisterPlayer(GameObject obj)
    {
        PlayerData p = new PlayerData();
        p.playerName = "Player_" + Random.Range(1000, 9999);
        p.playerObject = obj;
        p.startTime = Time.time;
        p.finished = false;

        players.Add(p);
    }

    public void PlayerFinished(GameObject obj)
    {
        PlayerData player = players.Find(p => p.playerObject == obj);

        if (player != null && !player.finished)
        {
            player.finishTime = Time.time - player.startTime;
            player.finished = true;

            Debug.Log(player.playerName + " terminou em " + player.finishTime);

            CheckEndRace();
        }
    }

    void CheckEndRace()
    {
        int finished = 0;

        foreach (var p in players)
        {
            if (p.finished) finished++;
        }

        if (finished == players.Count)
        {
            EndRace();
        }
    }

    void EndRace()
    {
        players.Sort((a, b) => a.finishTime.CompareTo(b.finishTime));

        Debug.Log("=== RESULTADO FINAL ===");

        for (int i = 0; i < players.Count; i++)
        {
            Debug.Log((i + 1) + "º - " + players[i].playerName + " - " + players[i].finishTime);
        }

        ShowPodium();
    }

    void ShowPodium()
    {
        if (players.Count < 3) return;

        MoveTo(players[0], podium1);
        MoveTo(players[1], podium2);
        MoveTo(players[2], podium3);

        Debug.Log("🥇 " + players[0].playerName);
        Debug.Log("🥈 " + players[1].playerName);
        Debug.Log("🥉 " + players[2].playerName);
    }

    void MoveTo(PlayerData player, Transform podium)
    {
        if (player.playerObject != null)
        {
            player.playerObject.transform.position = podium.position;
        }
    }
}

// ================= PLAYER DATA =================
[System.Serializable]
public class PlayerData
{
    public string playerName;
    public GameObject playerObject;
    public float startTime;
    public float finishTime;
    public bool finished;
}

// ================= CONTROLE DO CARRINHO =================
public class CartController : MonoBehaviour
{
    public float speed = 10f;
    public float turnSpeed = 120f;

    void Start()
    {
        SupermarketRace.instance.RegisterPlayer(this.gameObject);
        gameObject.tag = "Player";
    }

    void Update()
    {
        float move = Input.GetAxis("Vertical") * speed * Time.deltaTime;
        float turn = Input.GetAxis("Horizontal") * turnSpeed * Time.deltaTime;

        transform.Translate(0, 0, move);
        transform.Rotate(0, turn, 0);
    }

    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Finish"))
        {
            SupermarketRace.instance.PlayerFinished(this.gameObject);
        }
    }
}

// ================= NPC =================
public class NPCWalker : MonoBehaviour
{
    public Transform[] points;
    int current = 0;
    public float speed = 2f;

    void Update()
    {
        if (points.Length == 0) return;

        Transform target = points[current];
        transform.position = Vector3.MoveTowards(transform.position, target.position, speed * Time.deltaTime);

        if (Vector3.Distance(transform.position, target.position) < 0.2f)
        {
            current = (current + 1) % points.Length;
        }
    }
}
