- 👋 Hi, I’m @Oaris-Killer
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
Oaris-Killer/Oaris-Killer is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->#include "MyPlayerController.h"
#include "MyCharacter.h"
#include "MyProjectile.h"

AMyPlayerController::AMyPlayerController()
{
    bShowMouseCursor = true;
    DefaultMouseCursor = EMouseCursor::Crosshairs;
}

void AMyPlayerController::SetupInputComponent()
{
    Super::SetupInputComponent();

    InputComponent->BindAxis("MoveForward", this, &AMyPlayerController::MoveForward);
    InputComponent->BindAxis("MoveRight", this, &AMyPlayerController::MoveRight);

    InputComponent->BindAxis("Turn", this, &AMyPlayerController::AddYawInput);
    InputComponent->BindAxis("LookUp", this, &AMyPlayerController::AddPitchInput);

    InputComponent->BindAction("Jump", IE_Pressed, this, &AMyPlayerController::Jump);
    InputComponent->BindAction("Fire", IE_Pressed, this, &AMyPlayerController::Fire);
}

void AMyPlayerController::MoveForward(float Value)
{
    if (Value != 0.f)
    {
        APawn* const Pawn = GetPawn();
        if (Pawn)
        {
            const FVector Direction = FRotationMatrix(Pawn->GetActorRotation()).GetScaledAxis(EAxis::X);
            Pawn->AddMovementInput(Direction, Value);
        }
    }
}

void AMyPlayerController::MoveRight(float Value)
{
    if (Value != 0.f)
    {
        APawn* const Pawn = GetPawn();
        if (Pawn)
        {
            const FVector Direction = FRotationMatrix(Pawn->GetActorRotation()).GetScaledAxis(EAxis::Y);
            Pawn->AddMovementInput(Direction, Value);
        }
    }
}

void AMyPlayerController::Jump()
{
    APawn* const Pawn = GetPawn();
    if (Pawn)
    {
        AMyCharacter* const MyCharacter = Cast<AMyCharacter>(Pawn);
        if (MyCharacter)
        {
            MyCharacter->Jump();
        }
    }
}

void AMyPlayerController::Fire()
{
    APawn* const Pawn = GetPawn();
    if (Pawn)
    {
        AMyCharacter* const MyCharacter = Cast<AMyCharacter>(Pawn);
        if (MyCharacter)
        {
            const FVector StartLocation = MyCharacter->GetFirstPersonCameraComponent()->GetComponentLocation();
            const FVector EndLocation = StartLocation + MyCharacter->GetFirstPersonCameraComponent()->GetForwardVector() * 10000.f;

            FCollisionQueryParams CollisionParams;
            CollisionParams.AddIgnoredActor(MyCharacter);
            FHitResult HitResult;
            GetWorld()->LineTraceSingleByChannel(HitResult, StartLocation, EndLocation, ECollisionChannel::ECC_Visibility, CollisionParams);

            if (HitResult.bBlockingHit)
            {
                AMyProjectile* Projectile = GetWorld()->SpawnActor<AMyProjectile>(ProjectileClass, StartLocation, FRotationMatrix::MakeFromX(HitResult.ImpactNormal).Rotator());
                if (Projectile)
                {
                    const FVector LaunchDirection = (HitResult.ImpactPoint - Projectile->GetActorLocation()).GetSafeNormal();
                    Projectile->Launch(LaunchDirection);
                }
            }
            else
            {
                AMyProjectile* Projectile = GetWorld()->SpawnActor<AMyProjectile>(ProjectileClass, StartLocation, MyCharacter->GetActorRotation());
                if (Projectile)
                {
                    const FVector LaunchDirection = EndLocation.GetSafeNormal();
                    Projectile->Launch(LaunchDirection);
               

