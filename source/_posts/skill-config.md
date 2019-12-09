# 霸虎配置方法

##  Type
-   buff的逻辑

##  TypeOrder
-   区分相同Type的buff的依据，比如眩晕、冰冻的Type都是Set_Not_Action，眩晕标记为"stun"，冰冻标记为"frozen"
-   用来标记一些特殊的buff，比如技能里用TypeOrder来驱散相对应的buff
-   buff互顶的重要依据

##  一个buff成功添加需要经过的检测

1.  如果填了typerOrder，进行以下检测
    1.  身上已经有相同typeOrder的buff
        1.  如果新buff的数值大  ->  移除原buff，继续检测
        2.  如果新buff的数值小  ->  添加失败
        3.  如果数值相同
            1.  如果新buff回合数大  ->  移除原buff，继续检测
            2.  如果新buff回合数小  ->  添加失败
            2.  如果新buff回合相同  ->  添加失败
    2.  查看互斥表BuffRule2
        1.  通过    ->  继续检测
        2.  不通过  ->  添加失败
2.  配置表中Uniquely如果为1
    1.  身上如果已经有Uniquely为1的buff ->  移除原buff，继续检测
    2.  身上没有    ->  继续检测
3.  再次检测typerOrder如果填了，进行以下检测
    1.  查看互斥表BuffRule1
        1.  如果有需要被顶掉的buff  ->  移除需要被顶的buff，继续检测
        2.  如果没有要被顶掉的buff  ->  继续检测
4.  添加buff成功！！！

可以看出来，如果不填写typerOrder，那么buff很容易被加上去，一直加加加会导致数值爆炸的，千万注意

##  互斥表BuffRule1、BuffRule2的作用

-   举个栗子，新buff是"stun"，那么如果身上已经有"sleep"或者"enchantment"，那么"sleep"、"enchantment"都会被移除
    ```lua
    BuffRule1 = {
        frozen={"stun","sleep","enchantment","bleed","posion","fire"},
        stun={"sleep","enchantment"},
        sleep={"enchantment"}
    },
    ```
-   举个栗子，新buff是"sleep"，那么如果身上已经有"stun"或者"frozen"，那么新buff就无法添加了
    ```lua
    BuffRule2 = {
        enchantment={"frozen","stun","sleep"},
        sleep={"stun","frozen"},
        stun={"frozen"},
        poison={"frozen"},
        fire={"frozen"},
        bleed={"frozen"}
    },
    ```

##  Buff字段

-   <details><summary>加属性%：Add_Attri_Per</summary>
    
    物攻魔攻上升8%
    ```lua
    Type = "Add_Attri_Per",
    Custom = {"物理攻击","魔法攻击"},
    Value = 0.08,
    ```
    ```lua
    --程序实现：
    BuffScript["Add_Attri_Per"]  = {}
    BuffScript["Add_Attri_Per"].OnAdd = function(Buff)
        local attris = Buff:GetCustom()
        for i , attri in ipairs(attris) do
            Buff.Owner:AddMul(attri, Buff.Value)
        end
    end
    BuffScript["Add_Attri_Per"].OnRemove = function(Buff)
        local attris = Buff:GetCustom()
        for i , attri in ipairs(attris) do
            Buff.Owner:AddMul(attri, Buff.Value * -1)
        end
    end
    ```
    </details>

-   <details><summary>加属性：Add_Attri_Val</summary>
    
    物攻魔攻上升8点
    ```lua
    Type = "Add_Attri_Val",
    Custom = {"物理攻击","魔法攻击"},
    Value = 8,
    ```
    ```lua
    --程序实现：
    BuffScript["Add_Attri_Val"]  = {}
    BuffScript["Add_Attri_Val"].OnAdd = function(Buff)
        local attris = Buff:GetCustom()
        for i , attri in ipairs(attris) do
            Buff.Owner:Add(attri, Buff.Value)
        end
    end
    BuffScript["Add_Attri_Val"].OnRemove = function(Buff)
        local attris = Buff:GetCustom()
        for i , attri in ipairs(attris) do
            Buff.Owner:Add(attri, Buff.Value * -1)
        end
    end
    ```
    </details>

-   <details><summary>减属性%：Reduce_Attri_Per</summary>
    
    物攻魔攻下降8%
    ```lua
    Type = "Reduce_Attri_Per",
    Custom = {"物理攻击","魔法攻击"},
    Value = 0.08,
    ```
    ```lua
    --程序实现：
    BuffScript["Reduce_Attri_Per"]  = {}
    BuffScript["Reduce_Attri_Per"].OnAdd = function(Buff)
        local attris = Buff:GetCustom()
        for i , attri in ipairs(attris) do
            Buff.Owner:AddMul(attri, Buff.Value * -1)
        end
    end
    BuffScript["Reduce_Attri_Per"].OnRemove = function(Buff)
        local attris = Buff:GetCustom()
        for i , attri in ipairs(attris) do
            Buff.Owner:AddMul(attri, Buff.Value)
        end
    end
    ```
    </details>

-   <details><summary>减属性：Reduce_Attri_Val</summary>
    
    物攻魔攻下降8
    ```lua
    Type = "Reduce_Attri_Val",
    Custom = {"物理攻击","魔法攻击"},
    Value = 8,
    ```
    ```lua
    --程序实现：
    BuffScript["Reduce_Attri_Val"]  = {}
    BuffScript["Reduce_Attri_Val"].OnAdd = function(Buff)
        local attris = Buff:GetCustom()
        for i , attri in ipairs(attris) do
            Buff.Owner:Add(attri, Buff.Value * -1)
        end
    end
    BuffScript["Reduce_Attri_Val"].OnRemove = function(Buff)
        local attris = Buff:GetCustom()
        for i , attri in ipairs(attris) do
            Buff.Owner:Add(attri, Buff.Value)
        end
    end
    ```
    </details>

-   <details><summary>掉血Dot：Dot_Bleeding</summary>

    dot伤害为最大生命的1%
    ```lua
    Type = "Dot_Bleeding",
    Value = 0.01,
    ```
    ```lua
    --程序实现
    local max = Buff.Owner:Get("最大生命")
    local dmgHp = max * Buff.Value
    local dmg = Buff.Owner:CreateDamage()
    dmg.dmgType = EnumFight.DamageType.Dot
    dmg.dot = dmgHp
    Buff.Owner:DealDamage(dmg)
    if Buff.Round == 1 then
        Buff:RemoveSelf()
    end
    ```
    </details>
-   <details><summary>无法行动：Set_Not_Action</summary>

    ```lua
    Type = "Set_Not_Action",
    ```
    </details>
-   <details><summary>不屈盾：1HP_Shield</summary>
    
    接下来的4次伤害无效
    ```lua
    Type = "1HP_Shield",
    Value = 4,
    ```
    </details>

-   <details><summary>无敌盾：God_Shield</summary>
    
    伤害无效，并且免疫持续掉血Dot，无法行动
    ```lua
    Type = "God_Shield",
    ```
    </details>
-   <details><summary>律能免疫盾：Phy_Shield</summary>
    
    物理伤害无效
    ```lua
    Type = "Phy_Shield",
    ```
    </details>
-   <details><summary>辉能免疫盾：Mag_Shield</summary>
    
    魔法伤害无效
    ```lua
    Type = "Mag_Shield",
    ```
    </details>
-   <details><summary>律能吸收盾：Phy_Rc_Shield</summary>
    
    回复受到物理伤害的30%，并且物理伤害无效
    ```lua
    Type = "Phy_Rc_Shield",
    Value = 0.3
    ```
    </details>
-   <details><summary>辉能吸收盾：Mag_Rc_Shield</summary>
    
    回复受到魔法伤害的30%，并且魔法伤害无效
    ```lua
    Type = "Mag_Rc_Shield",
    Value = 0.3
    ```
    </details>
-   <details><summary>律能反射盾：Phy_Shield_Reflection</summary>
    
    反射受到物理伤害的30%，并且物理伤害无效
    ```lua
    Type = "Phy_Shield_Reflection",
    Value = 0.3
    ```
    </details>
-   <details><summary>辉能反射盾：Mag_Shield_Reflection</summary>
    
    反射受到魔法伤害的30%，并且魔法伤害无效
    ```lua
    Type = "Mag_Shield_Reflection",
    Value = 0.3
    ```
    </details>
-   <details><summary>暴击光环：Add_Group_CriticalRatio</summary>
    
    当有相同阵营的单位创建的时候，新建的单位增加5点暴击
    ```lua
    Type = "Add_Group_CriticalRatio",
    Value = 5
    ```
    ```lua
    Unit:Add("暴击率", Buff.Value)
    ```
    </details>

-   <details><summary>减伤光环：Add_Group_Dmg_Dec</summary>
    
    当有相同阵营的单位创建的时候，新建的单位增加5点减伤
    ```lua
    Type = "Add_Group_Dmg_Dec",
    Value = 5
    ```
    </details>
-   <details><summary>增伤光环：Add_Group_Dmg_Inc</summary>
    
    当有相同阵营的单位创建的时候，新建的单位增加5点增伤
    ```lua
    Type = "Add_Group_Dmg_Inc",
    Value = 5
    ```
    </details>
-   <details><summary>减攻光环：Reduce_Group_Attack_Per</summary>
    
    当有不同阵营的单位创建的时候，新建的单位减攻20%
    ```lua
    Type = "Reduce_Group_Attack_Per",
    Value = 0.2
    ```
    </details>
-   <details><summary>减防光环：Reduce_Group_Amor_Per</summary>
    
    当有不同阵营的单位创建的时候，新建的单位减防20%
    ```lua
    Type = "Reduce_Group_Amor_Per",
    Value = 0.2
    ```
    </details>
-   <details><summary>加血光环：Add_Group_Hp</summary>
    
    当有相同阵营的单位创建的时候，新建的单位增加最大血量血20%
    ```lua
    Type = "Add_Group_Hp",
    Value = 0.2
    ```
    </details>
-   <details><summary>红球连携攻击加成：Session_Red</summary>
    
    发动红球的时候，物理攻击、魔法攻击提升20%
    ```lua
    Type = "Session_Red",
    Value = 0.2
    ```
    </details>
-   <details><summary>蓝球连携攻击加成：Session_Blue</summary>
    
    发动蓝球的时候，物理攻击、魔法攻击提升20%
    ```lua
    Type = "Session_Blue",
    Value = 0.2
    ```
    </details>
-   <details><summary>黄球连携攻击加成：Session_Yellow</summary>
    
    发动黄球的时候，物理攻击、魔法攻击提升20%
    ```lua
    Type = "Session_Yellow",
    Value = 0.2
    ```
    </details>
-   <details><summary>黄球攻击触发回盾效果：Session_Yellow_Shield</summary>
    
    发动红球的时候，回复最大护盾20%的护盾
    ```lua
    Type = "Session_Yellow_Shield",
    Value = 0.2
    ```
    </details>
-   <details><summary>红球攻击提升暴击伤害：Session_Red_CriticalDamage</summary>
    
    发动红球的时候，暴击效果上升5点
    ```lua
    Type = "Session_Red_CriticalDamage",
    Value = 5
    ```
    </details>
-   <details><summary>蓝球攻击回血：Session_Blue_Heal</summary>
    
    发动蓝球的时候，治疗5点血
    ```lua
    Type = "Session_Blue_Heal",
    Value = 5
    ```
    </details>
-   <details><summary>免疫Buff：Ignore_Buff_Type</summary>
    
    免疫buff ， 可以是Type 也可以是TypeOrder
    ```lua
    Type = "Session_Blue_Heal",
    Custom = {"Set_Not_Action" , "Dot_Bleeding", "stun"},
    ```
    </details>
-   <details><summary>辉能易伤：Add_Mag_Dmg_Inc</summary>
    
    魔法攻击伤害增加1.1倍
    ```lua
    Type = "Add_Mag_Dmg_Inc",
    Value = 1.1
    ```
    </details>
-   <details><summary>律能易伤：Add_Phy_Dmg_Inc</summary>
    
    物理攻击伤害增加1.1倍
    ```lua
    Type = "Add_Phy_Dmg_Inc",
    Value = 1.1
    ```
    </details>
-   <details><summary>DOT易伤：Add_Dot_Dmg_Inc</summary>
    
    DOT伤害增加1.1倍
    ```lua
    Type = "Add_Dot_Dmg_Inc",
    Value = 1.1
    ```
    </details>
-   <details><summary>Buff易伤：Own_Buff_Dmg_Inc</summary>
    
    当攻击目标拥有任一表里的buff的时候，伤害提升1.1倍
    ```lua
    Type = "Own_Buff_Dmg_Inc",
    Custom = {"Set_Not_Action" , "Dot_Bleeding", "stun"},
    Value = 1.1
    ```
    </details>
-   <details><summary>易暴：Add_CriticalRatio_Inc</summary>
    
    受到伤害时，暴击几率上升5点
    ```lua
    Type = "Add_CriticalRatio_Inc",
    Value = 5
    ```
    </details>
-   <details><summary>盾衰减：Shield_Inc</summary>
    
    受到伤害时，受损护盾增加1.2倍
    ```lua
    Type = "Shield_Inc",
    Value = 1.2
    ```
    </details>
-   <details><summary>自爆：Self_Destruct</summary>
    
    3回合后自爆
    ```lua
    Type = "Self_Destruct",
    Round = 3
    ```
    </details>
-   <details><summary>嘲讽：Force_Target_Single</summary>
    
    嘲讽（单体目标，群攻无效）
    ```lua
    Type = "Force_Target_Single",
    ```
    </details>
-   <details><summary>黄球效果提升：Stack_Power_Increase</summary>
    
    黄球叠层数+1
    ```lua
    Type = "Stack_Power_Increase",
    Value = 1
    ```
    </details>
-   <details><summary>护盾衰减：Shield_ConsumeRatio_Change</summary>
    
    受击时，扣除1.1倍的护盾
    ```lua
    Type = "Shield_ConsumeRatio_Change",
    Value = 1.1
    ```
    </details>
-   <details><summary>攻击递增：Add_Atk_EveryRound</summary>
    
    每回合物理攻击、魔法攻击上升1.1倍
    ```lua
    Type = "Add_Atk_EveryRound",
    Value = 1.1
    ```
    </details>
-   <details><summary>重生：Im_Back</summary>
    
    死亡后复活，恢复30%血，生效后移除buff
    ```lua
    Type = "Im_Back",
    Value = 0.3
    ```
    </details>
-   <details><summary>顽强：No_Dead</summary>
    
    受到致命一击时依旧坚挺，并恢复1点血
    ```lua
    Type = "No_Dead",
    Value = 1
    ```
    </details>
-   <details><summary>顽强：No_Dead</summary>
    
    受到致命一击时依旧坚挺，并恢复1点血，生效后移除buff
    ```lua
    Type = "No_Dead",
    Value = 1
    ```
    </details>
-   <details><summary>蓄力：Force_Attack</summary>
    
    提高增伤2， 并且免疫流血、无法行动，直到下次攻击时移除效果
    ```lua
    Type = "Force_Attack",
    Value = 2
    ```
    </details>
-   <details><summary>禁止治疗：Disable_Heal</summary>
    
    ```lua
    Type = "Disable_Heal",
    ```
    </details>
-   <details><summary>提升被治疗量：Power_Heal</summary>
    
    被治疗的时候，提升1.2倍治疗效果
    ```lua
    Type = "Power_Heal",
    Value = 1.2,
    ```
    </details>
-   <details><summary>提升治疗量：Power_Heal_Source</summary>
    
    治疗的时候，提升1.2倍治疗效果
    ```lua
    Type = "Power_Heal_Source",
    Value = 1.2,
    ```
    </details>
-   <details><summary>技能封印：Disable_Skill</summary>
    
    ```lua
    Type = "Disable_Skill",
    ```
    </details>

-   <details><summary>运气光环1：Add_Buff_Pr_Cast</summary>
    
    提升自己释放ADD_BUFF_PR 10%的概率
    ```lua
    Type = "Add_Buff_Pr_Cast",
    Value = 0.1
    ```
    </details>
-   <details><summary>运气光环2：Add_Buff_Pr_Suffer</summary>
    
    降低自己受到ADD_BUFF_PR 10%的概率
    ```lua
    Type = "Add_Buff_Pr_Suffer",
    Value = 0.1
    ```
    </details>