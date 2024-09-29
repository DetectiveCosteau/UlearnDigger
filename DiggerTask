using System;
using Avalonia.Input;
using Digger.Architecture;

namespace Digger;

public class Terrain : ICreature
{
    public string GetImageFileName() => "Terrain.png";

    public int GetDrawingPriority() => 5;

    public CreatureCommand Act(int x, int y) => new();

    public bool DeadInConflict(ICreature conflictedObject)
    {
        return true;
    }
}

public class Player : ICreature
{
    public string GetImageFileName() => "Digger.png";

    public int GetDrawingPriority() => 3;

    public CreatureCommand Act(int x, int y)
    {
        var command = new CreatureCommand();
        switch (Game.KeyPressed)
        {
            case Key.Up:
                if (y - 1 >= 0 && !Tools.CheckPositionForCreatures(x, y - 1, typeof(Sack)))
                    command.DeltaY -= 1;
                break;
            case Key.Down:
                if (y + 1 < Game.MapHeight && !Tools.CheckPositionForCreatures(x, y + 1, typeof(Sack)))
                    command.DeltaY += 1;
                break;
            case Key.Left:
                if (x - 1 >= 0 && !Tools.CheckPositionForCreatures(x - 1, y, typeof(Sack)))
                    command.DeltaX -= 1;
                break;
            case Key.Right:
                if (x + 1 < Game.MapWidth && !Tools.CheckPositionForCreatures(x + 1, y, typeof(Sack)))
                    command.DeltaX += 1;
                break;
        }
        return command;
    }

    public bool DeadInConflict(ICreature conflictedObject)
    {
        return conflictedObject.GetType() == typeof(Sack)
            || conflictedObject.GetType() == typeof(Monster);
    }
}

public class Sack : ICreature
{
    private bool _isFalling;
    private int _fallingCounter;
    public string GetImageFileName() => "Sack.png";

    public int GetDrawingPriority() => 1;

    public CreatureCommand Act(int x, int y)
    {
        var command = new CreatureCommand();
        TryFall(command, x, y);
        if (!_isFalling && _fallingCounter >= 1)
            command.TransformTo = new Gold();
        return command;
    }

    public bool DeadInConflict(ICreature conflictedObject)
    {
        return false;
    }

    private bool IsProppedUp(int x, int y) => Tools
        .CheckPositionForCreatures(x, y + 1, typeof(Player), typeof(Monster))
        && !_isFalling;

    private CreatureCommand TryFall(CreatureCommand currentCommand, int x, int y)
    {
        var landingCreatures = new Type[] { typeof(Sack), typeof(Terrain), typeof(Gold) };
        if (y + 1 < Game.MapHeight
            && !Tools.CheckPositionForCreatures(x, y + 1, landingCreatures)
            && !IsProppedUp(x, y))
        {
            if (_isFalling)
                _fallingCounter++;
            currentCommand.DeltaY += 1;
            _isFalling = true;
        }
        else _isFalling = false;
        return currentCommand;
    }
}

public class Gold : ICreature
{
    public string GetImageFileName() => "Gold.png";

    public int GetDrawingPriority() => 4;

    public CreatureCommand Act(int x, int y) => new();

    public bool DeadInConflict(ICreature conflictedObject)
    {
        if (conflictedObject.GetType() == typeof(Player))
        {
            Game.Scores += 10;
            return true;
        }
        return conflictedObject.GetType() == typeof(Monster);
    }
}

public class Monster : ICreature
{
    public string GetImageFileName() => "Monster.png";

    public int GetDrawingPriority() => 2;

    public CreatureCommand Act(int x, int y)
    {
        var playerPosition = FindPlayer();
        if (playerPosition == null)
            return new CreatureCommand();
        return MoveToPlayer(playerPosition, x, y);
    }

    public bool DeadInConflict(ICreature conflictedObject)
    {
        return conflictedObject.GetType() == typeof(Monster)
            || conflictedObject.GetType() == typeof(Sack);
    }

    private int[] FindPlayer()
    {
        for (int x = 0; x < Game.MapWidth; x++)
        {
            for (int y = 0; y < Game.MapHeight; y++)
            {
                if (Game.Map[x, y] != null && Game.Map[x, y].GetType() == typeof(Player))
                    return new int[2] { x, y };
            }
        }
        return null;
    }

    private CreatureCommand MoveToPlayer(int[] playerPosition, int x, int y)
    {
        if (playerPosition[0] < x && CanMoveTo(x - 1, y))
            return new CreatureCommand() { DeltaX = -1 };
        if (playerPosition[1] < y && CanMoveTo(x, y - 1))
            return new CreatureCommand() { DeltaY = -1 };
        if (playerPosition[0] > x && CanMoveTo(x + 1, y))
            return new CreatureCommand() { DeltaX = 1 };
        if (playerPosition[1] > y && CanMoveTo(x, y + 1))
            return new CreatureCommand() { DeltaY = 1 };
        return new CreatureCommand();
    }

    private bool CanMoveTo(int x, int y)
    {
        return !Tools.CheckPositionForCreatures(x, y, typeof(Monster), typeof(Sack), typeof(Terrain));
    }
}

public class Tools
{
    public static bool CheckPositionForCreatures(int x, int y, params Type[] creaturesAsTypes)
    {
        foreach (var creatureType in creaturesAsTypes)
        {
            if (Game.Map[x, y] != null && Game.Map[x, y].GetType() == creatureType)
                return true;
        }
        return false;
    }
}
