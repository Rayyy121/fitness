using UnityEngine;

public class SquatController : MonoBehaviour
{
    [Header("關節物件綁定")]
    public Transform shinTransform;   // 小腿
    public Transform thighTransform;  // 大腿
    public Transform torsoTransform;  // 軀幹

    [Header("測試用感測器絕對角度 (相對於地面夾角)")]
    [Range(0f, 180f)]  public float shinSensorAngle = 90f;  
    [Range(0f, 180f)]  public float thighSensorAngle = 90f; 
    [Range(0f, 180f)]  public float torsoSensorAngle = 90f; 

    // 讓程式自己去算的骨架長度，不需要我們手動輸入了
    private float shinLength;  
    private float thighLength;

    void Start()
    {
        // === 關鍵核心：啟動時自動精準量測長度 ===
        // 依據你在編輯器上手動排好、對齊漂亮的初始位置，直接計算關節與關節間的絕對距離
        
        Vector3 footAnklePos = transform.position;        // 腳掌(腳踝)位置
        Vector3 shinKneePos = shinTransform.position;     // 小腿(膝蓋)位置
        Vector3 thighHipPos = thighTransform.position;    // 大腿(髖關節)位置

        // 1. 小腿長度 = 初始狀態下 [膝蓋] 到 [腳踝] 的距離
        shinLength = Vector3.Distance(shinKneePos, footAnklePos);

        // 2. 大腿長度 = 初始狀態下 [髖關節] 到 [膝蓋] 的距離
        thighLength = Vector3.Distance(thighHipPos, shinKneePos);
        
        // 印出 log 讓你知道程式幫你量出來的完美數字是多少
        Debug.Log($"[骨架自動對準成功] 偵測小腿長度: {shinLength}, 大腿長度: {thighLength}");
    }

    void Update()
    {
        // === 1. 旋轉角度套用 (方向修正) ===
        shinTransform.rotation = Quaternion.Euler(0, 0, 90f - shinSensorAngle);
        thighTransform.rotation = Quaternion.Euler(0, 0, 90f - thighSensorAngle);
        torsoTransform.rotation = Quaternion.Euler(0, 0, 90f - torsoSensorAngle);

        // === 2. 由上而下的精準關節連動 ===
        Vector3 footPos = transform.position; 

        float shinRad = shinSensorAngle * Mathf.Deg2Rad;
        float thighRad = thighSensorAngle * Mathf.Deg2Rad;

        // 由下往上算出「髖關節（屁股）」的精準位置
        Vector3 hipPos = new Vector3(
            footPos.x - (shinLength * Mathf.Cos(shinRad) + thighLength * Mathf.Cos(thighRad)),
            footPos.y + (shinLength * Mathf.Sin(shinRad) + thighLength * Mathf.Sin(thighRad)),
            footPos.z
        );

        // 各部位精準對位鎖死
        thighTransform.position = hipPos;
        torsoTransform.position = hipPos;

        // 從屁股往回推算膝蓋位置
        Vector3 kneePos = new Vector3(
            hipPos.x + thighLength * Mathf.Cos(thighRad),
            hipPos.y - thighLength * Mathf.Sin(thighRad),
            hipPos.z
        );





HipswaveTracker

using UnityEngine;
using UnityEngine.UI;
using System.Collections.Generic;

public class HipsWaveTracker : MonoBehaviour
{
    [Header("偵測對象")]
    public Transform hipsTransform;     
    public Transform footTransform;     

    [Header("UI 顯示物件")]
    public RectTransform waveBackground; 
    public RectTransform trackerPoint;   
    
    [Header("波形參數設定")]
    public float waveSpeed = 80f;        
    public float lineWidth = 3f; // 軌跡線條的粗細 (像素)
    
    [Header("圓滑度設定 (數字越小越圓滑，建議 2~5)")]
    public int smoothFactor = 3;

    private List<GameObject> segmentPool = new List<GameObject>();
    private List<Vector2> rawPoints = new List<Vector2>(); // 儲存原始點
    private Vector2 lastDrawnPoint;
    private bool isFirstPointInRound = true;
    
    private float currentX; 
    private float startX;   
    private float startY;     
    private float endX;
    
    private float maxHipsY;      
    private float minHipsY;      
    private float hipsMoveRange; 
    private float uiBottomY;     
    private float uiMoveRange;   
    private bool isInitialized = false;

    void Start()
    {
        if (waveBackground != null && trackerPoint != null)
        {
            trackerPoint.anchorMin = new Vector2(0.5f, 0.5f);
            trackerPoint.anchorMax = new Vector2(0.5f, 0.5f);
            trackerPoint.pivot = new Vector2(0.5f, 0.5f);

            startX = trackerPoint.localPosition.x; 
            startY = trackerPoint.localPosition.y; 
            currentX = startX; 

            endX = waveBackground.rect.width / 2f;    
            uiBottomY = -waveBackground.rect.height / 2f;
            uiMoveRange = startY - uiBottomY; 
        }
    }

    void Update()
    {
        if (hipsTransform == null || trackerPoint == null || waveBackground == null || footTransform == null) return;

        if (!isInitialized)
        {
            maxHipsY = hipsTransform.position.y; 
            minHipsY = footTransform.position.y; 
            hipsMoveRange = maxHipsY - minHipsY;
            if (hipsMoveRange <= 0) hipsMoveRange = 0.1f;
            
            isInitialized = true;
            return;
        }

        // X 軸前進
        currentX += Time.deltaTime * waveSpeed;

        // 走到最右邊觸底重播
        if (currentX > endX)
        {
            currentX = startX; 
            ClearAllSegments(); 
            rawPoints.Clear();
            isFirstPointInRound = true;
        }

        // 計算下蹲進度
        float squatProgress = (maxHipsY - hipsTransform.position.y) / hipsMoveRange;
        squatProgress = Mathf.Clamp01(squatProgress);

        float finalUIY = startY - (squatProgress * uiMoveRange);
        
        // 更新紅點位置
        trackerPoint.localPosition = new Vector3(currentX, finalUIY, 0f);

        // 記錄當前點的 UI 座標
        Vector2 currentPoint = trackerPoint.anchoredPosition;
        rawPoints.Add(currentPoint);

        // 當收集到足夠的點時，進行平滑化連線
        DrawSmoothLine();
    }

    // 核心演算法：利用 Chaikin 演算法或鄰近內插，讓曲線變得圓滑
    void DrawSmoothLine()
    {
        if (rawPoints.Count < 2) return;

        if (isFirstPointInRound)
        {
            lastDrawnPoint = rawPoints[0];
            isFirstPointInRound = false;
        }

        // 當新點增加時，透過內插在舊點與新點之間切出圓滑的過渡點
        Vector2 targetPoint = rawPoints[rawPoints.Count - 1];
        Vector2 prevPoint = rawPoints[rawPoints.Count - 2];

        // 根據平滑係數，將生硬的直線切碎成微小的弧線段
        int segments = smoothFactor;
        for (int i = 1; i <= segments; i++)
        {
            float t = (float)i / segments;
            // 透過 Lerp (線性內插) 配合前一幀的位置，達到柔化動態突變的效果
            Vector3 smoothedPoint = Vector2.Lerp(prevPoint, targetPoint, t);
            
            // 繪製微小圓滑片段
            CreateUISegment(lastDrawnPoint, smoothedPoint);
            lastDrawnPoint = smoothedPoint;
        }
    }

        shinTransform.position = kneePos;
    }
}
