import { NextResponse } from "next/server";

// In-memory store for demo — replace with Prisma in production
const activityLog = [
  { actionType: "pay_bill",       succeeded: true,  wasGuided: false, sandboxPracticedFirst: true,  timestamp: new Date(Date.now() - 1 * 86400000) },
  { actionType: "check_balance",  succeeded: true,  wasGuided: false, sandboxPracticedFirst: false, timestamp: new Date(Date.now() - 2 * 86400000) },
  { actionType: "make_call",      succeeded: true,  wasGuided: true,  sandboxPracticedFirst: false, timestamp: new Date(Date.now() - 3 * 86400000) },
  { actionType: "read_email",     succeeded: false, wasGuided: false, sandboxPracticedFirst: false, timestamp: new Date(Date.now() - 4 * 86400000) },
  { actionType: "pay_bill",       succeeded: true,  wasGuided: false, sandboxPracticedFirst: true,  timestamp: new Date(Date.now() - 5 * 86400000) },
];

const scamBlocksCount = 3;

function calculateConsistency(activities: typeof activityLog): number {
  const activeDays = new Set(
    activities.map(a => a.timestamp.toISOString().split("T")[0])
  );
  return Math.min(activeDays.size / 7, 1);
}

function getConfidenceLevel(score: number): string {
  if (score < 20) return "just_starting";
  if (score < 40) return "getting_comfortable";
  if (score < 60) return "growing_confident";
  if (score < 80) return "quite_independent";
  return "digital_champion";
}

export async function POST(request: Request) {
  try {
    const { userId } = await request.json();

    const activities = activityLog;

    const independentTasks = activities.filter(a => a.succeeded && !a.wasGuided).length;
    const totalTasks        = activities.length;
    const successRate       = totalTasks > 0 ? independentTasks / totalTasks : 0;
    const uniqueSkills      = new Set(activities.filter(a => a.succeeded).map(a => a.actionType)).size;
    const consistency       = calculateConsistency(activities);
    const sandboxBonus      = activities.some(a => a.sandboxPracticedFirst) ? 10 : 0;

    const score = Math.min(100, Math.round(
      (successRate * 40) +
      (scamBlocksCount > 0 ? 20 : 0) +
      (Math.min(uniqueSkills, 5) * 4) +
      (consistency * 10) +
      sandboxBonus
    ));

    const levelName = getConfidenceLevel(score);

    return NextResponse.json({
      score,
      levelName,
      userId,
      breakdown: { independentTasks, totalTasks, successRate, scamBlocksCount, uniqueSkills, consistency },
    });

  } catch (error) {
    console.error("Confidence calculation error:", error);
    return NextResponse.json({ error: "Failed to calculate confidence" }, { status: 500 });
  }
}

// Also support GET for easy dashboard polling
export async function GET() {
  const score = 72;
  return NextResponse.json({
    score,
    levelName: getConfidenceLevel(score),
    breakdown: { independentTasks: 3, totalTasks: 5, successRate: 0.6, scamBlocksCount, uniqueSkills: 4, consistency: 0.7 },
  });
}
