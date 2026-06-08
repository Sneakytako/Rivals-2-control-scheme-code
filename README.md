# Rivals-2-control-scheme-code
Rivals 2 control scheme code
JavaScript
/**
 * Updated Schema mapping to include a 14-bit patch number at the head.
 * This accommodates patch numbers from 0 up to 16,383 (perfect for 4-digit numbers).
 */
const SCHEMA = [
  { name: "patchNumber",           type: "integer", bits: 14 }, // Range: 0-9999 (Fits 4 digits)
  { name: "tapJump",               type: "boolean" },
  { name: "tapWallJump",           type: "boolean" },
  { name: "tapStrong",             type: "boolean" },
  { name: "rightStick",            type: "integer", bits: 3 },  // Range: 0-4 (5 values)
  { name: "leftStickDeadzone",     type: "integer", bits: 8 },  // Range: 0-249 (250 values)
  { name: "leftStickSensitivity",  type: "integer", bits: 8 },  // Range: 0-249 (250 values)
  { name: "hardPressThreshold",    type: "integer", bits: 6 },  // Range: 0-44 (45 values)
  { name: "rightStickThreshold",   type: "integer", bits: 7 },  // Range: 0-89 (90 values)
  { name: "rumble",                type: "boolean" },
  { name: "autoWalk",              type: "boolean" },
  { name: "doubleTapDash",         type: "boolean" },
  { name: "airGrabAirParry",       type: "integer", bits: 3 },  // Range: 0-6 (7 values)
  { name: "autoShorthopAerial",    type: "boolean" },
  { name: "noAirdodgeInTumble",    type: "boolean" },
  { name: "suppressDashAttack",    type: "boolean" },
  { name: "rollSetting",           type: "integer", bits: 2 },  // Range: 0-3 (4 values)
  { name: "wavedashAssist",        type: "boolean" },
  { name: "diAssist",              type: "boolean" },
  { name: "disableItemPickup",     type: "boolean" }
];

/**
 * Encodes the configuration object into a 12-digit ASCII code
 */
function encodeProfile(options) {
  const totalBits = SCHEMA.reduce((sum, item) => sum + (item.bits || 1), 0);
  const totalBytes = Math.ceil(totalBits / 8);
  const buffer = Buffer.alloc(totalBytes);

  let bitOffset = 0;

  for (const item of SCHEMA) {
    let value = options[item.name] ?? 0;
    if (item.type === "boolean") value = value ? 1 : 0;

    const bitsToWrite = item.bits || 1;

    for (let b = bitsToWrite - 1; b >= 0; b--) {
      const bit = (value >> b) & 1;
      const byteIdx = Math.floor(bitOffset / 8);
      const bitIdx = 7 - (bitOffset % 8);

      if (bit) {
        buffer[byteIdx] |= (1 << bitIdx);
      }
      bitOffset++;
    }
  }

  return buffer.toString('base64url');
}

/**
 * Decodes a 12-digit ASCII code back into your full controller configuration
 */
function decodeProfile(asciiStr) {
  const buffer = Buffer.from(asciiStr, 'base64url');
  const result = {};
  let bitOffset = 0;

  for (const item of SCHEMA) {
    const bitsToRead = item.bits || 1;
    let value = 0;

    for (let b = 0; b < bitsToRead; b++) {
      const byteIdx = Math.floor(bitOffset / 8);
      const bitIdx = 7 - (bitOffset % 8);

      const bit = (buffer[byteIdx] >> bitIdx) & 1;
      value = (value << 1) | bit;
      bitOffset++;
    }

    if (item.type === "boolean") {
      result[item.name] = value === 1;
    } else {
      result[item.name] = value;
    }
  }

  return result;
}

// ==========================================
// EXAMPLE RUN WITH OUTPUT DEMONSTRATION
// ==========================================

const sampleControllerConfig = {
  patchNumber: 2014,           // Tracks up to game update version 9999
  tapJump: true,
  tapWallJump: false,
  tapStrong: true,
  rightStick: 4,
  leftStickDeadzone: 125,
  leftStickSensitivity: 200,
  hardPressThreshold: 30,
  rightStickThreshold: 75,
  rumble: true,
  autoWalk: false,
  doubleTapDash: true,
  airGrabAirParry: 5,
  autoShorthopAerial: true,
  noAirdodgeInTumble: false,
  suppressDashAttack: true,
  rollSetting: 2,
  wavedashAssist: true,
  diAssist: false,
  disableItemPickup: true
};

console.log("--- Input Configuration ---");
console.log(sampleControllerConfig);

// Compress
const generatedCode = encodeProfile(sampleControllerConfig);
console.log(`\n--- Generated ASCII Code --- \n"${generatedCode}" (${generatedCode.length} digits)`);

// Reconstitute
const parsedConfig = decodeProfile(generatedCode);
console.log("\n--- Decoded Output ---");
console.log(parsedConfig);
