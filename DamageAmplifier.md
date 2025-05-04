using UnityEngine;

public class DamageAmplifier
{
    public enum AmplifierType
    {
        PLUS_CLICK_DAMAGE,
        CLICK_CRIT,
        PASSIVE_DAMAGE
    }

    public AmplifierType Type { get; private set; }
    public int Priority { get; private set; }
    public bool IsPassive { get; private set; }
    public float Value => InitValue + IncreasePerLevel * Mathf.Clamp(Level - 1, 0, int.MaxValue);
    public float InitValue { get; private set; }
    public float IncreasePerLevel { get; private set; }
    public int Price => InitPrice + IncreasePricePerLevel * Level;
    public int Level { get; private set; }
    public int InitPrice { get; private set; }
    public int IncreasePricePerLevel { get; private set; }
    public int Chance { get; private set; }

    public DamageAmplifier(AmplifierType type, int priority, bool isPassive,
        float value, float increase, int initPrice, int priceIncrease, int price, int chance = 100)
    {
        Type = type;
        Priority = priority;
        IsPassive = isPassive;
        InitValue = value;
        IncreasePerLevel = increase;
        InitPrice = price;
        IncreasePricePerLevel = priceIncrease;
        Chance = chance;

        Level = PlayerPrefs.GetInt("DA_" + Type.ToString(), 0);
    }

    public float CalculateDamage(float initDamage)
    {
        if (Level == 0)
            return initDamage;

        switch (Type)
        {
            case AmplifierType.PLUS_CLICK_DAMAGE:
            case AmplifierType.PASSIVE_DAMAGE:

                return initDamage + Value;

            case AmplifierType.CLICK_CRIT:

                if (Random.Range(0, 100) < Chance)
                    return initDamage + Value;
                else
                    return initDamage;

            default:
                return initDamage;
        }
    }

    public void LevelUp()
    {
        Level++;
        PlayerPrefs.SetInt("DA_" + Type.ToString(), Level);
    }
}
