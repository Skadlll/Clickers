using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using System;

public class Clicker : MonoBehaviour 
{
    public static Clicker Instance;

    public float Money
    {
        get => PlayerPrefs.GetFloat("Money", 0);
        private set => PlayerPrefs.SetFloat("Money", value);
    }

    [SerializeField]
    private Text money;
    [SerializeField]
    private List<AmplifierPerf> amplifierPerfs;

    private List<DamageAmplifier> amplifiers;

    private void Awake()
    {
        Instance = this;
    }


    void Start()
    {
        amplifiers = new List<DamageAmplifier>()
        {
            new DamageAmplifier(DamageAmplifier.AmplifierType.PLUS_CLICK_DAMAGE, 0, false , 2, 1.5f, 100, 100, 0),
            new DamageAmplifier(DamageAmplifier.AmplifierType.CLICK_CRIT, 100, false , 2,  2f,  200, 150, 25),
            new DamageAmplifier(DamageAmplifier.AmplifierType.PASSIVE_DAMAGE, 0, true , 2, 1.25f, 125, 250, 0),
        };

        for (int i = 0; i < amplifierPerfs.Count; i++)
            amplifierPerfs[i].SetData(amplifiers[i]);

        CalculateOfflineIncome();

        StartCoroutine(PassiveDamageDealer());
        UpdateUI();
    }

    private void OnApplicationQuit()
    {
        PlayerPrefs.SetString("LastPlayedTime", DateTime.UtcNow.ToString());
    }

    private void CalculateOfflineIncome()
    {
        string LastPlayedTimeString = PlayerPrefs.GetString("LastPlayedTime", null);
        if (LastPlayedTimeString == null)
            return;

        var LastPlayedTime = DateTime.Parse(LastPlayedTimeString);
        int timeSpanRestriction = 48 * 60 * 60;
        double secondsSpan = (DateTime.UtcNow - LastPlayedTime).TotalSeconds;

        if (secondsSpan > timeSpanRestriction)
            secondsSpan = timeSpanRestriction;

        float totalDamage = (float)secondsSpan * GetPassiveDamage();
        DamageTarget(totalDamage);
        Debug.Log($"Time span: {secondsSpan} Income: {totalDamage} ");
    }    

    private IEnumerator PassiveDamageDealer()
    {
        while (true)
        {
            yield return new WaitForSeconds(1);
            float damage = GetPassiveDamage();

            if (damage == 0)
                continue;
            
            DamageTarget(damage);
            EffectsController.Instance.CreatePassiveEffect((int)damage);
        }
    }

    public void Click()
    {
        float damage = GetClickDamage();
        DamageTarget(damage);
        EffectsController.Instance.CreateClickEffect((int)damage);
    }   
    
    private void DamageTarget(float damage)
    {
        AddMoney(damage);
    }

    private float GetClickDamage()
    {
        float damage = 1;

        var sortedAmplifiers = amplifiers.FindAll(x => !x.IsPassive);
        sortedAmplifiers.Sort((x, y) => x.Priority.CompareTo(y.Priority));

        foreach (var amplifier in sortedAmplifiers)
            damage = amplifier.CalculateDamage(damage);

        return damage;
    }

    private float GetPassiveDamage()
    {
        float damage = 0;

        var sortedAmplifiers = amplifiers.FindAll(x => x.IsPassive);
        sortedAmplifiers.Sort((x,y) => x.Priority.CompareTo(y.Priority));

        foreach (var amplifier in sortedAmplifiers)
            damage = amplifier.CalculateDamage(damage);

        return damage;
    }    

    public void UpdateUI()
    {
        money.text = "$" + (int)Money;
    }

    public void AddMoney(float value)
    {
        Money += value;
        UpdateUI();

        foreach (var pref in amplifierPerfs)
            pref.UpdateUI();
    }
}
